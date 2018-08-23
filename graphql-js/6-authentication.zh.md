---
title: Authentication
pageTitle: "Implementing Authentication in a GraphQL server with Node.js"
description: "Learn best practices for implementing authentication and authorization with Node.js, Express & Prisma."
question: Why was the 'User' type redefined in the application schema when it's already part of the Prisma database schema and could be imported from there?
answers: ["A 'User' type can never be imported because of how graphql-import works", "To hide potentially sensitive information from client applications", "This is important so users can reset their passwords", "It's a requirement from the GraphQL specification"]
correctAnswer: 1
---
在本节中,您将实现注册和登录功能,允许您的用户对您的GraphQL服务器进行身份验证. 

### 添加一个`User`输入您的数据模型

您需要的第一件事是在数据库中表示用户数据的方法. 你可以通过添加一个来实现`User`键入数据模型. 

你还想添加一个*关系*在. . 之间`User`并且已经存在`Link`用来表达的`Link`是的*发布*通过`User`秒. 

<Instruction>

打开`database/datamodel.graphql`并用以下内容替换其当前内容: 

```graphql{5,8-14}(path=".../hackernews-node/database/datamodel.graphql")
type Link {
  id: ID! @unique
  description: String!
  url: String!
  postedBy: User
}

type User {
  id: ID! @unique
  name: String!
  email: String! @unique
  password: String!
  links: [Link!]!
}
```

</Instruction>

你正在添加一个新的*关系领域*叫`postedBy`到了`Link`指向a的类型`User`实例. 该`User`类型然后有一个`links`这是一个列表的字段`Link`秒. 这就是您使用SDL表达一对多关系的方式. 

在对数据模型进行每次更改后,您需要重新部署Prisma服务以应用更改. 

<Instruction>

在项目的根目录中,运行以下命令: 

```bash(path=".../hackernews-node")
prisma deploy
```

</Instruction>

Prisma数据库架构`src/generated/prisma.graphql`随之而来的是Prisma服务的API已经更新. API现在也公开了CRUD操作`User`键入以及连接和断开连接的操作`User`和`Link`根据指定关系的元素. 

### 扩展应用程序架构

还记得架构驱动开发的过程吗?这一切都始于使用要添加到API的新操作扩展模式定义 - 在本例中为a`signup`和`login`突变. 

<Instruction>

在中打开应用程序架构`src/schema.graphql`并更新`Mutation`输入如下: 

```graphql{3,4}(path=".../hackernews-node/src/schema.graphql")
type Mutation {
  post(url: String!, description: String!): Link!
  signup(email: String!, password: String!, name: String!): AuthPayload
  login(email: String!, password: String!): AuthPayload
}
```

</Instruction>

接下来,继续添加`AuthPayload`随着一个`User`对文件的类型定义. 

<Instruction>

还在`src/schema.graphql`,添加以下类型定义: 

```graphql(path=".../hackernews-node/src/schema.graphql")
type AuthPayload {
  token: String
  user: User
}

type User {
  id: ID!
  name: String!
  email: String!
  links: [Link!]!
}
```

</Instruction>

所以,有效的`signup`和`login`突变表现非常相似. 两者都返回有关的信息`User`谁注册 (或登录) 以及`token`可用于验证针对API的后续请求. 这些信息捆绑在`AuthPayload`类型. 

但是等一下🤔为什么要重新定义`User`输入这个时间. 这不是也可以从Prisma数据库模式导入的类型吗?确实是!

但是,在这种情况下,您正在使用它*隐藏*某些信息`User`键入应用程序架构. 即,`password`字段 (虽然你将很快存储密码的哈希版本 - 所以即使它暴露在这里,客户也无法直接查询它) . 

### 实现解析器功能

使用新操作扩展模式定义后,需要为它们实现解析器函数. 在这样做之前,让我们实际重构一下你的代码,让它更加模块化!

您将每种类型的解析器拉出到自己的文件中. 

<Instruction>

首先,创建一个名为的新目录`resolvers`并向其添加三个文件: `Query.js`,`Mutation.js`和`AuthPayload.js`. 您可以使用以下命令执行此操作: 

```bash(path=".../hackernews-node")
mkdir src/resolvers
touch src/resolvers/Query.js
touch src/resolvers/Mutation.js
touch src/resolvers/AuthPayload.js
```

</Instruction>

接下来,移动执行`feed`解析器进入`Query.js`. 

<Instruction>

在`Query.js`,添加以下函数定义: 

```js(path=".../hackernews-node/src/resolvers/Query.js")
function feed(parent, args, context, info) {
  return context.db.query.links({}, info)
}

module.exports = {
  feed,
}
```

</Instruction>

这非常简单. 您只需使用不同文件中的专用功能重新实现之前的相同功能. 该`Mutation`旋转变压器是下一个. 

<Instruction>

打开`Mutation.js`并添加新的`login`和`signup`解析器 (你会添加`post`之后的解析器) : 

```js(path=".../hackernews-node/src/resolvers/Mutation.js")
async function signup(parent, args, context, info) {
  // 1
  const password = await bcrypt.hash(args.password, 10)
  // 2
  const user = await context.db.mutation.createUser({
    data: { ...args, password },
  }, `{ id }`)

  // 3
  const token = jwt.sign({ userId: user.id }, APP_SECRET)

  // 4
  return {
    token,
    user,
  }
}

async function login(parent, args, context, info) {
  // 1
  const user = await context.db.query.user({ where: { email: args.email } }, ` { id password } `)
  if (!user) {
    throw new Error('No such user found')
  }

  // 2
  const valid = await bcrypt.compare(args.password, user.password)
  if (!valid) {
    throw new Error('Invalid password')
  }

  const token = jwt.sign({ userId: user.id }, APP_SECRET)

  // 3
  return {
    token,
    user,
  }
}

module.exports = {
    signup,
    login,
    post,
}
```

</Instruction>

让我们再次使用好的'编号注释来了解这里发生了什么 - 从开始`signup`. 

1.  在里面`signup`变异,首先要做的是加密`User`使用密码的密码`bcryptjs`您将在稍后安装的库. 
2.  下一步是使用`Prisma`绑定实例来存储新的`User`在数据库中. 请注意,你正在硬编码`id`在选择集中 - 没有别的. 我们将很快详细讨论这个问题. 
3.  然后你生成一个用一个签名的JWT`APP_SECRET`. 你仍然需要创建它`APP_SECRET`并安装`jwt`这里使用的库. 
4.  最后,你回来了`token`和`user`. 

现在上了`login`突变: 

1.  代替*创建*一个新的`User`对象,你现在正在使用`Prisma`绑定实例来检索现有的`User`由...记录`email`在...中发送的地址`login`突变. 如果不`User`找到该电子邮件地址后,您将返回相应的错误. 请注意,这次你要求的是`id`和`password`在选择集中. 该`password`是必要的,因为它需要与提供的那个进行比较`login`突变. 
2.  下一步是将提供的密码与存储在数据库中的密码进行比较. 如果两者不匹配,您也会返回错误. 
3.  最后,你回来了`token`和`user`再次. 

两个解析器的实现相对简单 - 没什么太令人惊讶的. 现在唯一不清楚的是硬编码选择集只包含`id`领域. 如果传入的突变请求有关的更多信息会发生什么`User`?

### 添加`AuthPayload`分解器

例如,根据GraphQL架构定义考虑应该可能发生的突变: 

```graphql(nocopy)
mutation {
  login(
    email: "sarah@graph.cool"
    password: "graphql"
  ) {
    token
    user {
      id
      name
      links {
        url
        description
      }
    }
  }
}
```

这是一个正常的登录突变,其中有一些关于的信息`User`正在请求登录. 如何解决该突变的选择集?

使用当前的解析器实现,此突变实际上不会返回任何用户数据,因为所有这些都可以返回`User`是他们的`id` (因为这是Prisma要求的一切) . 解决这个问题的方法是实现额外的`AuthPayload`解析器并从那里的变异选择集中检索字段. 

<Instruction>

打开`AuthPayload.js`并向其添加以下代码: 

```js(path=".../hackernews-node/src/resolvers/AuthPayload.js")
function user(root, args, context, info) {
  return context.db.query.user({ where: { id: root.user.id } }, info)
}

module.exports = { user }
```

</Instruction>

这是传入选择集的位置`login`突变是*其实*已解决,并从数据库中检索请求的字段. 

> **注意**: 起初要理解这一点有点棘手. 要了解有关此主题的更多信息,请查看更详细的解释[这个](https://github.com/graphcool/prisma/issues/1737)GitHub问题并阅读[这个](https://blog.graph.cool/graphql-server-basics-demystifying-the-info-argument-in-graphql-resolvers-6f26249f613a)博客文章关于`info`目的. 

现在让我们来完成实施. 

<Instruction>

首先,将所需的依赖项添加到项目中: 

```bash(path=".../hackernews-node/")
yarn add jsonwebtoken bcryptjs
```

</Instruction>

接下来,您将创建一些在几个地方重用的实用程序. 

<Instruction>

在里面创建一个新文件`src`目录并调用它`utils.js`: 

```bash(path=".../hackernews-node/")
touch src/utils.js
```

</Instruction>

<Instruction>

现在,将以下代码粘贴到其中: 

```js(path=".../hackernews-node/src/utils.js")
const jwt = require('jsonwebtoken')
const APP_SECRET = 'GraphQL-is-aw3some'

function getUserId(context) {
  const Authorization = context.request.get('Authorization')
  if (Authorization) {
    const token = Authorization.replace('Bearer ', '')
    const { userId } = jwt.verify(token, APP_SECRET)
    return userId
  }

  throw new Error('Not authenticated')
}

module.exports = {
  APP_SECRET,
  getUserId,
}
```

</Instruction>

该`APP_SECRET`用于签署您为用户签发的JWT. 它完全独立于`secret`在...中指定的`prisma.yml`. 实际上,它与Prisma完全没有关系,即如果你要更换数据库层的实现,`APP_SECRET`将继续以完全相同的方式使用. 

该`getUserId`function是一个辅助函数,你将在需要验证的解析器中调用 (例如`post`) . 它首先检索`Authorization`标题 (包含`User`来自传入的HTTP请求的JWT) . 然后它验证JWT并从中检索用户的ID. 请注意,如果该进程因任何原因未成功,则该函数将抛出一个*例外*. 因此,您可以使用它来"保护"需要身份验证的解析程序. 

最后,您需要将所有内容导入`Mutation.js`. 

<Instruction>

将以下import语句添加到顶部`Mutation.js`: 

```js(path=".../hackernews-node/src/resolvers/Mutation.js")
const bcrypt = require('bcryptjs')
const jwt = require('jsonwebtoken')
const { APP_SECRET, getUserId } = require('../utils')
```

</Instruction>

### 需要身份验证`post`突变

在您要测试身份验证流程之前,请确保完成架构/解析程序设置. 现在`post`解析器仍然缺失. 

<Instruction>

在`Mutation.js`,添加以下解析器实现`post`: 

```js(path=".../hackernews-node/src/resolvers/Mutation.js")
function post(parent, args, context, info) {
  const userId = getUserId(context)
  return context.db.mutation.createLink(
    {
      data: {
        url: args.url,
        description: args.description,
        postedBy: { connect: { id: userId } },
      },
    },
    info,
  )
}
```

</Instruction>

与之前的实施相比,实施中有两件事情发生了变化`index.js`: 

1.  你现在正在使用`getUserId`函数检索的ID`User`存储在设置为的JWT中`Authorization`传入HTTP请求的标头. 因此,你知道哪个`User`正在创造`Link`这里. 回想一下,检索不成功`userId`将导致异常并且函数范围在退出之前退出`createLink`调用变异. 
2.  你当时也在使用它`userId`至*连*该`Link`与...一起创建`User`谁在创造它. 这是通过发生的[`connect`](https://www.prisma.io/docs/reference/prisma-api/mutations-ol0yuoz6go#overview)-突变. 

真棒!您现在需要做的最后一件事是使用新的解析器实现`index.js`. 

<Instruction>

打开`index.js`并导入现在包含文件顶部的解析器的模块: 

```js(path=".../hackernews-node/src/index.js")
const Query = require('./resolvers/Query')
const Mutation = require('./resolvers/Mutation')
const AuthPayload = require('./resolvers/AuthPayload')
```

</Instruction>

<Instruction>

然后,更新的定义`resolvers`对象看起来如下: 

```js(path=".../hackernews-node/src/index.js")
const resolvers = {
  Query,
  Mutation,
  AuthPayload
}
```

</Instruction>

就是这样,您已准备好测试身份验证流程!🔓

### 测试身份验证流程

你要做的第一件事就是测试`signup`突变从而创造一个新的`User`在数据库中. 

<Instruction>

如果您还没有这样做,请先停止并重启服务器,然后再将其删除**CTRL + C**然后跑`node src/index.js`. 然后,通过运行使用GraphQL CLI打开一个新的Playground`graphql playground`. 

</Instruction>

请注意,如果您仍然打开它,您可以从之前"重新使用"您的Playground  - 重要的是您重新启动服务器,以便实际应用您对实现所做的更改. 

<Instruction>

现在,发送以下变异来创建一个新的`User`: 

```graphql
mutation {
  signup(
    name: "Alice"
    email: "alice@graph.cool"
    password: "graphql"
  ) {
    token
    user {
      id
    }
  }
}
```

</Instruction>

<Instruction>

从服务器的响应中,复制身份验证`token`并在Playground中打开另一个选项卡. 在新标签内,打开**HTTP HEADERS**左下角的窗格并指定`Authorization`标题 - 类似于您之前使用Prisma Playground所做的事情. 再次,替换`__TOKEN__`使用实际令牌: 

```json
{
  "Authorization": "Bearer __TOKEN__"
}
```

</Instruction>

每当您现在从该选项卡发送查询/变异时,它将携带身份验证令牌. 

<Instruction>

随着`Authorization`标头就位,将以下内容发送到您的GraphQL服务器: 

```graphql
mutation {
  post(
    url: "www.graphql-europe.org"
    description: "Europe's biggest GraphQL conference"
  ) {
    id
  }
}
```

</Instruction>

![](https://imgur.com/jEToi1S.png)

当您的服务器收到此突变时,它会调用`post`解析器因此验证提供的JWT. 另外,新的`Link`创建的现在已连接到`User`您之前发送过的`signup`突变. 

要验证一切正常,您可以发送以下内容`login`突变: 

```graphql
mutation {
  login(
    email: "alice@graph.cool"
    password: "graphql"
  ) {
    token
    user {
      email
      links {
        url
        description
      }
    }
  }
}
```
