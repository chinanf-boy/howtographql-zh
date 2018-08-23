---
title: Realtime GraphQL Subscriptions
pageTitle: "Realtime GraphQL Subscriptions with Node.JS Tutorial"
description: "Learn how to implement GraphQL subscriptions with Node.js, Express & Prisma to add realtime functionality to an app."
question: Which of the following statements is true?
answers: ["The 'node' field of a subscription is always null for CREATED-mutations", "The 'previousValues' field of a subscription is always null for DELETED-mutations", "The 'previousValues' field of a subscription is always null for UPDATED-mutations", "The 'node' field of a subscription is always null for DELETED-mutations"]
correctAnswer: 3
---
在本节中,您将学习如何通过实现GraphQL订阅将实时功能引入您的应用程序. 目标是实现GraphQL服务器公开的两个订阅: 

-   在新的时候向订阅的客户发送实时更新`Link`元素是*创建*
-   在存在时向订阅的客户端发送实时更新`Link`元素是*upvoted*

### 什么是GraphQL订阅?

订阅是GraphQL功能,允许服务器在特定时间向其客户端发送数据*事件*发生. 订阅通常用[的WebSockets](https://en.wikipedia.org/wiki/WebSocket). 在该设置中,服务器与其订阅的客户端保持稳定的连接. 这也打破了用于以前与API交互的"请求 - 响应周期". 

相反,客户端最初通过发送一个来打开与服务器的长期连接*订阅查询*指定哪个*事件*它感兴趣. 每次发生此特定事件时,服务器都使用连接将事件数据推送到订阅的客户端. 与Prisma订阅

### 幸运的是,Prisma为订阅提供了开箱即用的支持. 

事实上,如果你看一下Prisma架构`src/generated/prisma.graphql`,你会注意到的`Subscription`类型已经存在,目前看起来如下: 

```graphql(path=".../hackernews-node/src/generated/prisma.graphql"&nocopy)
type Subscription {
  link(where: LinkSubscriptionWhereInput): LinkSubscriptionPayload
  user(where: UserSubscriptionWhereInput): UserSubscriptionPayload
}
```

这些订阅可以触发以下内容*事件*: 

-   一个新节点是**创建**
-   现有节点**更新**
-   现有节点是**删除**

请注意,您可以限制订阅应该触发的事件. 例如,如果您只想订阅对一个特定的更新`Link`或者当具体的`User`删除,您也可以通过提供`where`订阅查询的参数. 

GraphQL订阅遵循与查询和突变相同的语法,因此您可以订阅*事件*现有的`Link`使用以下订阅更新元素: 

```graphql(nocopy)
subscription {
  link(where: {
    mutation_in: [UPDATED]
  }) {
    node {
      id
      url
      description
    }
  }
}
```

每次现有订阅时都会触发此订阅`Link`更新,服务器将发送 (可能更新) `url`和`description`更新的值`Link`. 

我们也快速考虑一下`LinkSubscriptionPayload`来自`src/generated/prisma.graphql`: 

```graphql(nocopy)
type LinkSubscriptionPayload {
  mutation: MutationType!
  node: Link
  updatedFields: [String!]
  previousValues: LinkPreviousValues
}
```

以下是各个字段的用途: 

#### `mutation: MutationType!`

`MutationType`是一个`enum`有三个值: 

```graphql(nocopy)
enum MutationType {
  CREATED
  UPDATED
  DELETED
}
```

该`mutation`在球场上`LinkSubscriptionPayload`因此,类型携带信息发生了什么样的突变. 

#### `node: Link`

这个字段代表了`Link`已创建,更新或删除的元素,并允许检索有关它的更多信息. 

请注意`DELETED`-mutations,`node`一直会`null`!如果您需要了解有关的更多详细信息`Link`删除后,你可以使用`previousValues`相反 (更多关于那个) . 

> **注意**: 一个术语*节点*有时在GraphQL中用于引用单个元素. 节点本质上对应于数据库中的记录. 

#### `updatedFields: [String!]`

您可能感兴趣的一条信息`UPDATED`-mutations是哪个*领域*已在变异中更新. 这就是这个领域的用途. 

假设客户已订阅以下订阅查询: 

```graphql(nocopy)
subscription {
  link {
    updatedFields
  }
}
```

现在,假设服务器收到以下突变: 

```graphql(nocopy)
mutation {
  updateLink(
    where: {
      id: "..."
    }
    data: {
      description: "An even greater website"
    }
  )
}
```

订阅的客户端将接收以下有效负载: 

```json(nocopy)
{
  "data": {
    "link": {
      "updatedFields": ["description"]
    }
  }
}
```

这是因为突变只更新了`Link`的`description`领域 - 不是`url`. 

#### `previousValues: LinkPreviousValues`

该`LinkPreviousValues`类型看起来非常相似`Link`本身: 

```graphql(nocopy)
type LinkPreviousValues {
  id: ID!
  description: String!
  url: String!
}
```

它基本上是一个辅助类型,它反映了字段`Link`. 

`previousValues`仅用于`UPDATED`- 和`DELETED`-mutations. 对于`CREATED`-mutations,它永远是`null` (出于同样的原因`node`为null`DELETED`-mutations) . 

#### 将所有东西放在一起

考虑样本`updateLink`- 再次从上一节开始. 但我们现在假设,订阅查询包括我们刚刚讨论过的所有字段: 

```graphql(nocopy)
subscription {
  link {
    mutation
    updatedFields
    node {
      url
      description
    }
    previousValues {
      url
      description
    }
  }
}
```

以下是服务器在执行突变后推送到客户端的有效负载: 

```json
{
  "data": {
    "link": {
      "mutation": "UPDATED",
      "updatedFields": ["description"],
      "node": {
        "url": "www.example.org",
        "description": "An even greater website"
      },
      "previousValues": {
        "url": "www.example.org",
        "description": "A great website"
      }
    }
  }
}
```

请注意,这假定已更新`Link`在进行突变之前具有以下值: 

-   `url`: `www.example.org`
-   `description`: `A great website`

### 订阅新的`Link`分子

足够的谈话,更多的编码!让我们实现允许您的客户订阅新创建的订阅`Link`元素. 

就像查询和突变一样,实现订阅的第一步是扩展GraphQL架构定义. 

<Instruction>

打开您的应用程序架构并添加`Subscription`类型: 

```graphql(path=".../hackernews-node/src/schema.graphql)
type Subscription {
  newLink: LinkSubscriptionPayload
}
```

</Instruction>

<Instruction>

因为你正在引用`LinkSubscriptionPayload`从Prisma架构中,您还需要在文件顶部调整import语句并导入该类型: 

```graphql(path=".../hackernews-node/src/schema.graphql)
# import Link, LinkSubscriptionPayload from "./generated/prisma.graphql"
```

</Instruction>

接下来,继续执行解析器`newLink`领域. 订阅的解析器与查询和突变的解析器略有不同: 

1.  他们返回的不是直接返回任何数据`AsyncIterator`随后由GraphQL服务器用于将事件数据推送到客户端. 
2.  订阅解析器包装在对象内部,需要作为a的值提供`subscribe`领域. 

<Instruction>

要遵循解析器实现的模块化结构,首先要创建一个名为的新文件`Subscription.js`: 

```bash(path=".../hackernews-node)
touch src/resolvers/Subscription.js
```

</Instruction>

<Instruction>

以下是在新文件中实现订阅解析程序的方法: 

```js
function newLinkSubscribe (parent, args, context, info) {
  return context.db.subscription.link(
    { where: { mutation_in: ['CREATED'] } },
    info,
  )
}

const newLink = {
  subscribe: newLinkSubscribe
}

module.exports = {
  newLink,
}
```

</Instruction>

代码似乎非常简单. 如前所述,订阅解析程序是作为a的值提供的`subscribe`普通JavaScript对象中的字段. 

该`Prisma`绑定实例`context`也揭露了一个`subscription`代理来自Prisma GraphQL API的订阅的对象. 此函数用于解析订阅并将事件数据推送到订阅的客户端. Prisma正在处理引擎盖下处理实时功能的所有复杂性. 

<Instruction>

打开`index.js`并导入文件顶部的订阅模块: 

```js(path=".../hackernews-node/src/index.js")
const Query = require('./resolvers/Query')
const Mutation = require('./resolvers/Mutation')
const AuthPayload = require('./resolvers/AuthPayload')
const Subscription = require('./resolvers/Subscription')
```

</Instruction>

<Instruction>

然后,更新的定义`resolvers`对象看起来如下: 

```js(path=".../hackernews-node/src/index.js")
const resolvers = {
  Query,
  Mutation,
  AuthPayload,
  Subscription,
}
```

</Instruction>

### 测试订阅

有了所有代码,就可以测试实时API了. 您可以通过一次使用GraphQL Playground的两个实例 (即窗口) 来实现. 

<Instruction>

如果您还没有,请先重新启动服务器**CTRL + C**然后跑`node src/index.js`再次. 

</Instruction>

<Instruction>

接下来,打开两个浏览器窗口并导航到GraphQL服务器的端点: [`http://localhost:4000`](http://localhost:4000). 

</Instruction>

正如您可能猜到的那样,您将使用一个Playground发送订阅查询,从而创建与服务器的永久websocket连接. 第二个将用于发送`post`触发订阅的突变. 

<Instruction>

在一个Playground中,发送以下订阅: 

```graphql
subscription {
  newLink {
    node {
      id
      url
      description
      postedBy {
        id
        name
        email
      }
    }
  }
}
```

</Instruction>

与发送查询和突变时发生的情况相反,您不会立即看到操作的结果. 相反,有一个加载微调器指示它正在等待事件发生. 

![](https://imgur.com/wSkSXZE.png)

是时候触发订阅事件了. 

<Instruction>

发送以下内容`post`GraphQL Playground中的变异. 请记住,您需要对其进行身份验证 (请参阅上一章以了解其工作原理) : 

```graphql
mutation {
  post(
    url: "www.graphqlweekly.com"
    description: "Curated GraphQL content coming to your email inbox every Friday"
  ) {
    id
  }
}
```

</Instruction>

现在观察正在运行订阅的Playground: 

![](https://imgur.com/V89gYLE.png)

### 添加投票功能

要添加的下一个功能是投票功能,可以让用户使用*给予好评*某些链接. 这里的第一步是扩展您的Prisma数据模型以表示投票. 

<Instruction>

打开`database/datamodel.graphql`并添加它看起来如下: 

```graphql{7,16,19-23}(path=".../hackernews-node/database/datamodel.graphql")
type Link {
  id: ID! @unique
  createdAt: DateTime!
  description: String!
  url: String!
  postedBy: User
  votes: [Vote!]!
}

type User {
  id: ID! @unique
  name: String!
  email: String! @unique
  password: String!
  links: [Link!]!
  votes: [Vote!]!
}

type Vote {
  id: ID! @unique
  link: Link!
  user: User!
}
```

</Instruction>

如您所见,您添加了一个新的`Vote`键入数据模型. 它与之有一对多的关系`User`和`Link`类型. 

要应用更改并更新Prisma GraphQL API,以便它包含新的CRUD操作`Vote`类型,您需要再次部署服务. 

<Instruction>

在终端中运行以下命令: 

```bash(path=".../hackernews-node)
prisma deploy
```

</Instruction>

现在,考虑到模式驱动开发的过程,继续并扩展应用程序模式的模式定义,以便GraphQL服务器也公开`vote`突变: 

```graphql{5}(path=".../hackernews-node/src/schema.graphql")
type Mutation {
  post(url: String!, description: String!): Link!
  signup(email: String!, password: String!, name: String!): AuthPayload
  login(email: String!, password: String!): AuthPayload
  vote(linkId: ID!): Vote
}
```

<Instruction>

引用`Vote`还需要从Prisma数据库模式导入类型: 

```graphql(path=".../hackernews-node/src/schema.graphql")
# import Link, LinkSubscriptionPayload, Vote from "./generated/prisma.graphql"
```

</Instruction>

你知道接下来会发生什么: 实现相应的解析器功能. 

<Instruction>

添加以下功能`src/resolvers/Mutation.js`: 

```js(path=".../hackernews-node/src/resolvers/Mutation.js")
async function vote(parent, args, context, info) {
  // 1
  const userId = getUserId(context)

  // 2
  const linkExists = await context.db.exists.Vote({
    user: { id: userId },
    link: { id: args.linkId },
  })
  if (linkExists) {
    throw new Error(`Already voted for link: ${args.linkId}`)
  }

  // 3
  return context.db.mutation.createVote(
    {
      data: {
        user: { connect: { id: userId } },
        link: { connect: { id: args.linkId } },
      },
    },
    info,
  )
}
```

</Instruction>

这是正在发生的事情: 

1.  类似于你在做什么`post`解析器,第一步是验证传入的JWT`getUserId`辅助功能. 如果它有效,该函数将返回`userId`的`User`谁在提出要求. 如果JWT无效,该函数将抛出异常. 
2.  该`db.exists.Vote(...)`函数调用对你来说是新的. 该`Prisma`绑定对象不仅公开镜像Prisma数据库模式中的查询,突变和预订的函数. 它也会生成一个`exists`数据模型中每种类型的函数. 该`exists`功能需要一个`where`过滤器对象,允许指定有关该类型元素的特定条件. 只有在条件适用的情况下*至少*数据库中的一个元素,`exists`功能返回`true`. 在这种情况下,您正在使用它来验证请求`User`尚未投票赞成`Link`这是由...确定的`args.linkId`. 
3.  如果`exists`回报`false`,`createVote`将用于创建一个新的`Vote`那个元素*连接的*到了`User`和`Link`. 

<Instruction>

另外,不要忘记调整export语句以包含`vote`模块中的解析器: 

```js{5}(path=".../hackernews-node/src/resolvers/Mutation.js")
module.exports = {
  post,
  signup,
  login,
  vote,
}
```

</Instruction>

本章的最后一项任务是添加一个在新的时候触发的订阅`Vote`正在创建. 你将使用类似的方法`newLink`查询. 

<Instruction>

添加一个新字段到`Subscription`应用程序架构的类型: 

```graphql{3}(path=".../hackernews-node/src/schema.graphql")
type Subscription {
  newLink: LinkSubscriptionPayload
  newVote: VoteSubscriptionPayload
}
```

</Instruction>

<Instruction>

接下来,导入`VoteSubscriptionPayload`从Prisma API的GraphQL架构到应用程序架构: 

```graphql(path=".../hackernews-node/src/schema.graphql")
# import Link, LinkSubscriptionPayload, Vote, VoteSubscriptionPayload from "./generated/prisma.graphql"
```

</Instruction>

最后,您需要添加订阅解析器功能. 

<Instruction>

将以下代码添加到`Subscription.js`: 

```js(path=".../hackernews-node/src/resolvers/Subscription.js")
function newVoteSubscribe (parent, args, context, info) {
  return context.db.subscription.vote(
    { where: { mutation_in: ['CREATED'] } },
    info,
  )
}

const newVote = {
  subscribe: newVoteSubscribe
}
```

</Instruction>

<Instruction>

并相应地更新文件的export语句: 

```js{3}(path=".../hackernews-node/src/resolvers/Subscription.js")
module.exports = {
  newLink,
  newVote,
}
```

</Instruction>

好吧,就是这样!您现在可以测试您的实施`newVote`订阅. 

您可以使用以下订阅: 

```graphql
subscription {
  newVote {
    node {
      link {
        url
        description
      }
      user {
        name
        email
      }
    }
  }
}
```

如果你不确定自己写一个,这是一个样本`vote`你可以使用突变. 你需要更换`__LINK_ID__`占位符与`id`一个实际的`Link`来自您的数据库. 此外,请确保在发送突变时进行身份验证. 

```graphql
mutation {
  vote(linkId: "__LINK_ID__") {
    link {
      url
      description
    }
    user {
      name
      email
    }
  }
}
```

![](https://imgur.com/ii2X4Yc.png)
