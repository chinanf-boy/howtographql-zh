---
title: Getting Started
pageTitle: "Getting Started with GraphQL, Node.js and Prisma Tutorial"
description: "Learn how to setup a GraphQL server with Node.js & Express as well as best practices for defining the GraphQL schema."
question: "What role do the root fields play for a GraphQL API?"
answers: ["The three root fields are: Query, Mutation and Subscription", "Root fields implement the available API operations", "Root fields define the available API operations", "Root field is another term for resolver"]
correctAnswer: 2
---
在本节中,您将为GraphQL服务器设置项目并实现第一个GraphQL查询. 最后,我们将谈论理论并了解[GraphQL架构](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e). 

### 创建项目

本教程教您如何从头开始构建GraphQL服务器,因此您需要做的第一件事就是创建一个保存GraphQL服务器文件的目录!

<Instruction>

打开终端,导航到您选择的位置并运行以下命令: 

```bash
mkdir hackernews-node
cd hackernews-node
yarn init -y
```

</Instruction>

> **注意**本教程使用[纱](https://yarnpkg.com/en/)管理你的项目. 如果你喜欢使用[npm](https://www.npmjs.com/),您可以使用简单地运行等效命令`npm`. 

这将创建一个名为的新目录`hackernews-node`并用a初始化它`package.json`文件. `package.json`是您正在构建的Node应用程序的配置文件. 它列出了所有依赖项和其他配置选项 (例如*脚本*) 应用程序需要. 

### 创建原始GraphQL服务器

使用项目目录,您可以继续为GraphQL服务器创建入口点. 这将是一个名为的文件`index.js`,位于名为的目录内`src`. 

<Instruction>

在您的终端中,首先创建`src`目录然后空`index.js`文件: 

```bash(path=".../hackernews-node/")
mkdir src
touch src/index.js
```

</Instruction>

> **注意**: 上面的代码块使用目录名称进行注释. 它表明*哪里*你需要执行terminal命令. 

要启动应用程序,您现在可以执行`node src/index.js`在 - 的里面`hackernews-node`目录. 目前,这不会做任何事情,因为`index.js`仍然是空的\\\_ (ツ) \_ /¯

让我们开始构建GraphQL服务器!您需要做的第一件事是 - 惊喜 - 为项目添加依赖项. 

<Instruction>

在终端中运行以下命令: 

```bash(path=".../hackernews-node/")
yarn add graphql-yoga
```

</Instruction>

[`graphql-yoga`](https://github.com/graphcool/graphql-yoga)是一个功能齐全的GraphQL服务器. 它基于[Express.js](https://expressjs.com/)以及一些其他库来帮助您构建生产就绪的GraphQL服务器. 

以下是其功能列表: 

-   GraphQL规范兼容
-   支持文件上传
-   GraphQL订阅的实时功能
-   适用于TypeScript类型
-   对GraphQL Playground的开箱即用支持
-   通过Express中间件可扩展
-   解决GraphQL架构中的自定义指令
-   查询性能跟踪
-   接受两者`application/json`和`application/graphql`内容类型
-   到处运行: 可以通过部署`now`,`up`,AWS Lambda,Heroku等. 

完美,是时候写一些代码🙌

<Instruction>

打开`src/index.js`并键入以下内容: 

```js(path="../hackernews-node/src/index.js")
const { GraphQLServer } = require('graphql-yoga')

// 1
const typeDefs = `
type Query {
  info: String!
}
`

// 2
const resolvers = {
  Query: {
    info: () => `This is the API of a Hackernews Clone`
  }
}

// 3
const server = new GraphQLServer({
  typeDefs,
  resolvers,
})
server.start(() => console.log(`Server is running on http://localhost:4000`))
```

</Instruction>

> **注意**: 此代码块使用文件名注释. 它指示您需要将所显示的代码放入哪个文件. 注释还链接到GitHub上的相应文件,以帮助您弄清楚*哪里*在文件中你需要把它放在你不确定的情况下. 此外,虽然代码块有一个用于复制代码的按钮,但我们鼓励您自己键入所有内容,因为这会大大改善您的学习体验. 

好的,让我们通过阅读编号的评论来了解这里发生了什么: 

1.  该`typeDefs`常数定义你的*GraphQL架构* (更多关于这一点) . 在这里,它定义了一个简单的`Query`用一个打字*领域*叫`info`. 该字段具有类型`String!`. 类型定义中的感叹号表示该字段永远不会`null`..
2.  该`resolvers`对象是实际的*履行*GraphQL架构. 注意它的结构如何与里面的类型定义的结构相同`typeDefs`: `Query.info`. 
3.  最后,模式和解析器被捆绑并传递给`GraphQLServer`这是从中导入的`graphql-yoga`. 这告诉服务器接受了哪些API操作以及如何解析它们. 

继续测试您的GraphQL服务器!

### 测试GraphQL服务器

<Instruction>

在项目的根目录中,运行以下命令: 

```bash(path=".../hackernews-node/")
node src/index.js
```

</Instruction>

如终端输出所示,服务器现在正在运行`http://localhost:4000`. 要测试服务器的API,请打开浏览器并导航到该URL. 

你会看到的是一个[GraphQL游乐场](https://github.com/graphcool/graphql-playground),一个功能强大的"GraphQL IDE",可让您以交互方式探索API的功能. 

![](https://imgur.com/9RC6x9S.png)

点击绿色**SCHEMA**- 右侧的按钮,您可以打开API文档. 此文档是根据您的架构定义自动生成的,并显示架构的所有API操作和数据类型. 

![](https://imgur.com/81Ho6YM.png)

我们去发送你的第一个GraphQL查询. 在左侧的编辑器窗格中键入以下内容: 

```graphql
query {
  info
}
```

现在通过单击将查询发送到服务器**玩**- 中间的按钮 (或使用键盘快捷键**CMD + Enter**) . 

![](https://imgur.com/EnW3HE5.png)

恭喜,您刚刚实施并成功测试了您的第一个GraphQL查询🎉

现在,记得我们谈到的时候的定义`info: String!`字段并说感叹号意味着这个领域永远不会`null`. 好吧,既然您正在实施解析器,那么您可以控制该字段的值,对吧?那么,如果你回来会发生什么`null`而不是解析器实现中的实际信息字符串?随意试试吧!

在`index.js`,更新定义`resolvers`如下: 

```js{3}(path=".../hackernews-node/src/index.js")
const resolvers = {
  Query: {
    info: () => null,
  }
}
```

要测试结果,您需要重新启动服务器: 首先,使用它来停止它**CTRL + C**在键盘上,然后通过运行重新启动它`node src/index.js`再次. 

现在,再次发送查询. 这次,它返回一个错误: `Error: Cannot return null for non-nullable field Query.info.`

![](https://imgur.com/VLVE5Vv.png)

这里发生的是潜在的[`graphql-js`](https://github.com/graphql/graphql-js/)参考实现可确保解析器的返回类型符合GraphQL架构中的类型定义. 换句话说,它可以保护您免于犯下愚蠢的错误!

这实际上是GraphQL的核心优势之一: 它强制API实际上以模式定义所承诺的方式运行!这样,每个有权访问GraphQL架构的人都可以100%确定API返回的API操作和数据结构. 

### 关于GraphQL架构的一个词

每个GraphQL API的核心都有一个GraphQL架构. 所以,让我们快速谈谈它. 

> **注意**: 在本教程中,我们只讨论本主题的表面. 如果您想更深入一些并了解有关GraphQL架构及其在GraphQL API中的角色的更多信息,请务必查看[这个](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e)优秀的文章. 

GraphQL模式通常用GraphQL编写[模式定义语言](https://blog.graph.cool/graphql-sdl-schema-definition-language-6755bcb9ce51) (SDL) . SDL有一个类型系统,允许定义数据结构 (就像其他强类型编程语言,如Java,TypeScript,Swift,Go,...) . 

但是,这对于为GraphQL服务器定义API有何帮助?每个GraphQL架构都有三个特殊的*根类型*,这些被称为`Query`,`Mutation`和`Subscription`. 根类型对应于GraphQL提供的三种操作类型: 查询,突变和订阅. 调用这些根类型上的字段*根田*并定义可用的API操作. 

作为示例,请考虑我们上面使用的简单GraphQL架构: 

```graphql(nocopy)
type Query {
  info: String!
}
```

此架构只有一个根域,称为`info`. 在发送对GraphQL API的查询,突变或订阅时,这些始终需要以root字段开头!在这种情况下,我们只有一个根字段,因此实际上只有一个可以被API接受的查询. 

我们现在考虑一个稍微更高级的例子: 

```(nocopy)
type Query {
  users: [User!]!
  user(id: ID!): User
}

type Mutation {
  createUser(name: String!): User!
}

type User {
  id: ID!
  name: String!
}
```

在这种情况下,我们有三个根域: `users`和`user`上`Query`以及`createUser`上`Mutation`. 附加的定义`User`type是必需的,否则架构定义将不完整. 

可以从此架构定义派生的API操作是什么?好吧,我们知道每个API操作总是需要以root字段开头. 但是,我们尚未了解它的样子*类型*根域的本身就是另一个[对象类型](http://graphql.org/learn/schema/#object-types-and-fields). 这是这里的情况,根域的类型是`[User!]!`,`User`和`User!`. 在里面`info`以前的例子,根字段的类型是a`String`,这是一个[标量类型](http://graphql.org/learn/schema/#scalar-types). 

-   `typeDefs`: 这些是从您导入的应用程序架构中的类型定义`src/schema.graphql`. 
-   `resolvers`: 这是一个镜像的JavaScript对象`Query`,`Mutation`和`Subscription`应用程序架构中的类型及其字段. 应用程序模式中的每个字段由该对象中具有相同名称的函数表示. 
-   `context`: 这是一个通过解析器链传递的对象,每个解析器都可以读取或写入. 

当根字段的类型是对象类型时,您可以使用该对象类型的字段进一步扩展查询 (或变异/订阅) . 扩展部分被称为*选择集*. 

以下是实现上述模式的GraphQL API接受的操作: 

```graphql(nocopy)
# Query for all users
query {
  users {
    id
    name
  }
}

# Query a single user by their id
query {
  user(id: "user-1") {
    id
    name
  }
}

# Create a new user
mutation {
  createUser(name: "Bob") {
    id
    name
  }
}
```

有几点需要注意: 

-   在这些例子中,我们总是查询`id`和`name`返回的`User`对象. 我们可能会省略其中任何一个. 但请注意,在查询对象类型时,您需要在选择集中至少查询其中一个字段. 
-   对于选择集中的字段,根字段的类型是否正常无关紧要*需要*或者a*名单*. 在上面的示例模式中,三个根字段都有不同[类型修饰符](http://graphql.org/learn/schema/#lists-and-non-null) (即,列表和/或要求的不同组合) `User`类型: 
    -   为了`users`字段,返回类型`[User!]!`意味着它返回一个*名单* (这本身不可能`null`) 的`User`元素. 该列表也可以不包含元素`null`. 因此,您始终可以保证接收空列表或仅包含非空的列表`User`对象. 
    -   为了`user(id: ID!)`字段,返回类型`User`表示返回的值可能是`null` *要么*一个`User`目的. 
    -   为了`createUser(name: String!)`字段,返回类型`User!`表示此操作始终返回a`User`目的. 

当您提供此信息时,`Prisma`instance将获得对数据库服务的完全访问权限,并可用于稍后解决传入请求. 

Phew,足够的理论😠让我们再去写一些代码吧!
