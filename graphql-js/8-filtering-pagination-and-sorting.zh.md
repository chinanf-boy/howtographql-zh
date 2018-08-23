---
title: Filtering, Pagination & Sorting
pageTitle: "GraphQL Filtering, Pagination & Sorting Tutorial with JavaScript"
description: "Learn how to add filtering and pagination capabilities to a GraphQL API with Node.js, Express & Prisma."
question: Which arguments are typically used to paginate through a list in the Prisma API using limit-offset pagination?
answers: ["skip & last", "skip & first", "first & last", "where & orderBy"]
correctAnswer: 1
---
这是本教程的最后一部分,您将在其中实现API的最后润色. 目标是允许客户限制列表`Link`由...返回的元素`feed`通过提供过滤和分页参数进行查询. 

### 过滤

感谢Prisma,您将能够毫不费力地为您的API实现过滤功能. 与前几章类似,强大的Prisma引擎将执行查询解析的繁重工作. 您需要做的就是将传入的查询委托给它. 

第一步是考虑要通过API公开的过滤器. 在你的情况下,`feed`您的API中的查询将接受a*过滤字符串*. 然后查询应该只返回`Link`元素在哪里`url` *要么*该`description` *包含*过滤字符串. 

<Instruction>

继续添加`filter`字符串到`feed`查询应用程序架构: 

```graphql{3}(path=".../hackernews-node/src/schema.graphql)
type Query {
  info: String!
  feed(filter: String): [Link!]!
}
```

</Instruction>

接下来,您需要更新该实现`feed`解析器以解释客户端可以提供的新参数. 

<Instruction>

打开`src/resolvers/Query.js`并更新`feed`解析器看起来如下: 

```js{2-11}(path=".../hackernews-node/src/resolvers/Query.js")
function feed(parent, args, context, info) {
  const where = args.filter
    ? {
        OR: [
          { url_contains: args.filter },
          { description_contains: args.filter },
        ],
      }
    : {}

  return context.db.query.links({ where }, info)
}
```

</Instruction>

如果不`filter`提供了字符串,然后是`where`对象将只是一个空对象,并且Prisma引擎在返回响应时不会应用过滤条件`links`查询. 

万一有`filter`来自传入`args`,你正在构建一个`where`从上面表达我们的两个过滤条件的对象. 这个`where`Prisma使用参数来过滤掉那些参数`Link`不符合指定条件的元素. 

请注意`Prisma`绑定对象将上面的函数调用转换为看起来有点类似的GraphQL查询[这个](https://blog.graph.cool/graphql-server-basics-demystifying-the-info-argument-in-graphql-resolvers-6f26249f613a). 此查询由Yoga服务器发送到Prisma API并在那里解决: 

```graphql(nocopy)
query ($filter: String) {
  links(where: {
    OR: [{ url_contains: $filter }, { description_contains: $filter }]
  }) {
    id
    url
    description
  }
}
```

> **注意**: 您可以在中了解有关Prisma过滤功能的更多信息[文档](https://www.prisma.io/docs/reference/prisma-api/queries-ahwee4zaey#filtering-by-field). 探索这些的另一种方法是使用Prisma API的Playground并阅读该模式的文档`where`通过使用Playground的自动完成功能直接进行参数或实验. ![](https://imgur.com/PjyLATP.png)

这已经是过滤功能了!继续测试您的过滤器API  - 这是您可以使用的示例查询: 

```graphql
query {
  feed(filter: "Prisma") {
    id
    description
    url
  }
}
```

![](https://imgur.com/1EQsPKc.png)

### 分页

分页是API设计中一个棘手的话题. 在高层次上,有两种主要方法可以解决它: 

-   **限价偏移**: 请求具体*块*列表中提供要检索的项目的索引 (实际上,您主要提供起始索引 (*抵消*) 以及要检索的项目数 (*限制*) ) . 
-   **基于游标**: 这个分页模型有点先进. 列表中的每个元素都与一个唯一的ID相关联*光标*) . 通过列表分页的客户端然后提供起始元素的光标以及要检索的项目的计数. 

Prisma支持两种分页方法 (请阅读更多内容) [文档](https://www.prisma.io/docs/reference/prisma-api/queries-ahwee4zaey#pagination)) . 在本教程中,您将实现限制偏移分页. 

> **注意**: 您可以阅读有关这两种分页方法背后的想法的更多信息[这里](https://dev-blog.apollodata.com/understanding-pagination-rest-graphql-and-relay-b10f835549e7). 

在Prisma API中,限制和偏移的调用方式不同: 

-   该*限制*叫做`first`,这意味着你抓住了*第一*提供的起始索引后的x个元素. 请注意,您还有一个`last`可用的参数相应地返回*持续*x元素. 
-   该*开始索引*叫做`skip`,因为在收集要返回的项目之前,您正在跳过列表中的许多元素. 如果`skip`没有提供,是的`0`默认. 然后,分页始终从列表的开头开始 (或者在您使用时结束`last`) . 

所以,继续添加`skip`和`first`争论的`feed`查询. 

<Instruction>

打开您的应用程序架构并调整`feed`查询接受`skip`和`first`参数: 

```graphql{3}(path=".../hackernews-node/src/schema.graphql)
type Query {
  info: String!
  feed(filter: String, skip: Int, first: Int): [Link!]!
}
```

</Instruction>

现在,转到解析器实现. 

<Instruction>

在`src/resolvers/Query.js`,调整执行`feed`解析: 

```js{12}(path=".../hackernews-node/src/resolvers/Query.js")
function feed(parent, args, context, info) {
  const where = args.filter
    ? {
        OR: [
          { url_contains: args.filter },
          { description_contains: args.filter },
        ],
      }
    : {}

  return context.db.query.links(
    { where, skip: args.skip, first: args.first },
    info,
  )
}
```

</Instruction>

真正所有这一切在这里改变的是`links`查询现在接收两个可能由传入承载的附加参数`args`目的. 再一次,Prisma将为我们做出艰苦的努力

您可以使用以下返回第二个查询的查询来测试分页API`Link`从列表中: 

```graphql
query {
  feed(
    first: 1
    skip: 1
  ) {
    id
    description
    url
  }
}
```

![](https://imgur.com/fhkKct7.png)

### 排序

使用Prisma,可以返回已排序的元素列表 (*有序*) 根据具体标准. 例如,您可以订购列表`Link`按字母顺序排列`url`要么`description`. 对于Hackernews API,您将把它留给客户端来决定它应该如何排序,从而包括瑜伽服务器API中Prisma API的所有排序选项. 您可以通过直接引用来实现`LinkOrderByInput`来自Prisma数据库模式的枚举. 这是它的样子: 

```graphql(path=".../hackernews-node/src/generated/prisma.graphql"&nocopy)
enum LinkOrderByInput {
  id_ASC
  id_DESC
  description_ASC
  description_DESC
  url_ASC
  url_DESC
  updatedAt_ASC
  updatedAt_DESC
  createdAt_ASC
  createdAt_DESC
}
```

它代表了列表的各种方式`Link`元素可以排序. 

<Instruction>

打开应用程序架构并导入`LinkOrderByInput`来自Prisma数据库模式的枚举: 

```graphql(path=".../hackernews-node/src/schema.graphql")
# import Link, LinkSubscriptionPayload, Vote, VoteSubscriptionPayload, LinkOrderByInput from "./generated/prisma.graphql"
```

</Instruction>

<Instruction>

然后,调整`feed`再次查询包括`orderBy`论据: 

```graphql{3}(path=".../hackernews-node/src/schema.graphql")
type Query {
  info: String!
  feed(filter: String, skip: Int, first: Int, orderBy: LinkOrderByInput): [Link!]!
}
```

</Instruction>

解析器的实现类似于您刚刚使用分页API执行的操作. 

<Instruction>

更新的实现`feed`解析器`src/resolvers/Query.js`并通过`orderBy`对Prisma的争论: 

```js{12}(path=".../hackernews-node/src/resolvers/Query.js")
function feed(parent, args, context, info) {
  const where = args.filter
    ? {
        OR: [
          { url_contains: args.filter },
          { description_contains: args.filter },
        ],
      }
    : {}

  return context.db.query.links(
    { where, skip: args.skip, first: args.first, orderBy: args.orderBy },
    info,
  )
}
```

</Instruction>

真棒!这是一个查询,按创建日期对返回的链接进行排序: 

```graphql
query {
  feed(orderBy: createdAt_ASC) {
    id
    description
    url
  }
}
```

### 退还总金额`Link`分子

您要为Hackernews API实现的最后一件事就是信息*多少* `Link`元素当前存储在数据库中. 要做到这一点,你要重构`feed`查询一下并创建一个由API返回的新类型: `Feed`. 

<Instruction>

添加新的`Feed`键入您的应用程序架构. 然后还调整了返回类型`feed`查询相应: 

```graphql{3,6-9}(path=".../hackernews-node/src/schema.graphql")
type Query {
  info: String!
  feed(filter: String, skip: Int, first: Int, orderBy: LinkOrderByInput): Feed!
}

type Feed {
  links: [Link!]!
  count: Int!
}
```

</Instruction>

<Instruction>

现在,继续调整`feed`再次解析: 

```js{12-15,18-25,29-30}(path=".../hackernews-node/src/resolvers/Query.js")
async function feed(parent, args, context, info) {
  const where = args.filter
    ? {
        OR: [
          { url_contains: args.filter },
          { description_contains: args.filter },
        ],
      }
    : {}

  // 1
  const queriedLinks = await context.db.query.links(
    { where, skip: args.skip, first: args.first, orderBy: args.orderBy },
    `{ id }`,
  )

  // 2
  const countSelectionSet = `
    {
      aggregate {
        count
      }
    }
  `
  const linksConnection = await context.db.query.linksConnection({}, countSelectionSet)

  // 3
  return {
    count: linksConnection.aggregate.count,
    linkIds: queriedLinks.map(link => link.id),
  }
}
```

</Instruction>

1.  您首先使用提供的过滤,排序和分页参数来检索许多`Link`元素. 这与您之前所做的类似,不同之处在于您现在再次对查询的选择集进行硬编码并仅检索`id`被查询的领域`Link`秒. 事实上,如果你试图通过`info`这里,API会抛出一个错误 (读取[这个](https://blog.graph.cool/graphql-server-basics-demystifying-the-info-argument-in-graphql-resolvers-6f26249f613a)文章了解原因) . 
2.  接下来,你正在使用`linksConnection`从Prisma数据库模式查询以检索总数`Link`当前存储在数据库中的元素. 你正在硬编码选择集来返回`count`可以通过的方式检索的元素`aggregate`领域. 
3.  该`count`可以直接退货. 该`links`在. 上指定的`Feed`类型尚未返回 - 这只会发生在您仍需要实现的下一个解析程序级别. 请注意,因为你正在返回`linkIds`从此解析程序级别,解析程序链中的下一个解析程序将可以访问这些解析程序. 

现在的最后一步是实现解析器`Feed`类型. 

<Instruction>

在里面创建一个新文件`src/resolvers`并称之为`Feed.js`: 

```bash(path=".../hackernews-node)
touch src/resolvers/Feed.js
```

</Instruction>

<Instruction>

现在,添加以下代码: 

```js(path=".../hackernews-node/src/resolvers/Feed.js")
function links(parent, args, context, info) {
  return context.db.query.links({ where: { id_in: parent.linkIds } }, info)
}

module.exports = {
  links,
}
```

</Instruction>

这是哪里的`links`来自的领域`Feed`类型*其实*得到解决. 如你所见,传入`parent`争论正在进行`linkIds`这是在之前的解析程序级别返回的. 

最后一步是在实例化时包含新的解析器`GraphQLServer`. 

<Instruction>

在`index.js`,先进口`Feed`解析器位于文件顶部: 

```js(path=".../hackernews-node/src/index.js")
const Feed = require('./resolvers/Feed')
```

</Instruction>

<Instruction>

然后,将其包含在`resolvers`目的: 

```js{6}(path=".../hackernews-node/src/index.js")
const resolvers = {
  Query,
  Mutation,
  AuthPayload,
  Subscription,
  Feed
}
```

</Instruction>

你现在可以测试改版了`feed`查询如下: 

```graphql
query {
  feed {
    count
    links {
      id
      description
      url
    }
  }
}
```

![](https://imgur.com/x2z3lLU.png)
