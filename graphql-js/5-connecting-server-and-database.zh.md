---
title: Connecting Server and Database with Prisma Bindings
pageTitle: "Connecting a Database to a GraphQL Server with Prisma Tutorial"
description: "Learn how to add a database to your GraphQL server. The database is powered by Prisma and connected to the server via GraphQL bindings."
question: "What is the main responsibility of the Prisma binding instance that's attached to the 'context'?"
answers: ["Expose the application schema to client applications", "Translate the GraphQL operations from the Prisma API into JavaScript functions", "Translate the GraphQL operations from the application layer into JavaScript functions", "Generate SQL queries"]
correctAnswer: 1
---
在本节中,您将要将GraphQL服务器与[Prisma](https://www.prisma.io)数据库服务,它为您的数据库提供接口. 连接通过实现[Prisma绑定](https://github.com/graphcool/prisma-binding). 

### 更新解析程序函数以使用Prisma绑定

您将通过一些清理和重构来开始本节. 

<Instruction>

打开`index.js`并完全删除`links`数组以及`idCount`变量 - 您不再需要它们,因为数据现在将存储在实际数据库中. 

</Instruction>

接下来,您需要更新解析器函数的实现,因为它们现在正在访问不存在的变量. 此外,您现在希望从数据库返回实际数据,而不是本地虚拟数据. 

<Instruction>

还在`index.js`,更新`resolvers`对象看起来如下: 

```js{4-6,8-17}(path=".../hackernews-node/src/index.js")
const resolvers = {
  Query: {
    info: () => `This is the API of a Hackernews Clone`,
    feed: (root, args, context, info) => {
      return context.db.query.links({}, info)
    },
  },
  Mutation: {
    post: (root, args, context, info) => {
      return context.db.mutation.createLink({
        data: {
          url: args.url,
          description: args.description,
        },
      }, info)
    },
  },
}
```

</Instruction>

哇,这看起来很奇怪!有一堆新的东西正在发生,让我们试着了解发生了什么,从开始`feed`解析器. 

#### 该`context`和`info`参数

以前,`feed`解析器没有采取任何论据 - 现在收到了*四*. 事实上,前两个是不需要的. 但`context`和`info`是. 

还记得我们之前说的所有GraphQL解析器函数*总是*收到四个论点. 现在你知道最后两个被调用了什么 - 但它们用于什么?

该`context`argument是一个JavaScript对象,解析器链中的每个解析器都可以读取和写入 - 因此它基本上是解析器进行通信的一种方法. 正如您稍后会看到的那样,在初始化GraphQL服务器本身时,也可以写入它. 所以,它也是一种方式*您*将任意数据或函数传递给解析器. 在这种情况下,你将附上这个`db`反对`context`- 更多关于这一点. 

该`info`对象携带有关传入的GraphQL查询的信息 (以. 的形式) [查询AST](https://medium.com/@cjoudrey/life-of-a-graphql-query-lexing-parsing-ca7c5045fad8)) . 例如,它知道在查询的选择集中请求了哪些字段. 

> **注意**: 要更深入地了解解析器参数,请阅读以下两篇文章: 
>
> -   [GraphQL Server基础: 架构](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e)
> -   [GraphQL Server基础知识: 揭开神秘面纱`info`GraphQL解析器中的参数](https://blog.graph.cool/graphql-server-basics-demystifying-the-info-argument-in-graphql-resolvers-6f26249f613a)

现在您已经基本了解了传递给解析器的参数,让我们看看它们是如何在解析器函数的实现中使用的. 

#### 理解`feed`分解器

该`feed`解析器实现如下: 

```js(path=".../hackernews-node/src/index.js"&nocopy)
feed: (root, args, context, info) => {
  return context.db.query.links({}, info)
},
```

它访问一个`db`对象`context`. 正如您将在稍后看到的那样`db`对象实际上是一个`Prisma`来自的绑定实例[`prisma-binding`](https://github.com/graphcool/prisma-binding)NPM包. 

这个`Prisma`绑定实例有效地将Prisma数据库模式转换为可以调用的JavaScript函数. 在调用这样的函数时,`Prisma`绑定实例将汇总一个GraphQL查询并将其发送给Prisma API. 但是你传递给函数的两个参数是什么?

第一个参数包含您可能希望与查询一起提交的任何变量. 因为你不需要任何变量`links`现在查询,它只是一个空对象. 

第二个参数由. 使用`Prisma`绑定实例来生成*选择集*对于它发送到Prisma API的查询. 你之前学到了`info`object携带有关传入查询的选择集的信息. 这里发生的事情是你基本上*转发* (即[委托](https://blog.graph.cool/graphql-schema-stitching-explained-schema-delegation-4c6caf468405)) 传入查询到Prisma引擎. 该`info`对象需要传递,以便Prisma知道传入查询中请求的字段. 

事实上,第二个论点也可能是一个*串*包含查询的选择集. 考虑这个示例函数调用: 

```js(nocopy)
const selectionSet = `
{
  id
  description
  url
}
`
context.db.query.links({}, selectionSet)
```

这将对应于发送到Prisma API的以下GraphQL查询: 

```graphql(nocopy)
query {
  links {
    id
    description
    url
  }
}
```

#### 理解`post`分解器

该`post`解析器现在看起来像这样: 

```js(path=".../hackernews-node/src/index.js"&nocopy)
post: (root, args, context, info) => {
  return context.db.mutation.createLink({
    data: {
      url: args.url,
      description: args.description,
    },
  }, info)
},
```

类似于`feed`解析器,你只需要调用一个函数`Prisma`附加到的绑定实例`context`. 

回想一下`Prisma`绑定实例本质上将Prisma数据库模式转换为可以从JavaScript调用的函数. 调用这些函数会将相应的查询/变异发送到底层的Prisma API. 

在这种情况下,你发送了`createLink`来自Prisma的GraphQL架构的变异. 作为参数,您将传递解析器通过的数据`args`参数. 

同样,函数调用的第二个参数是`info`包含传入突变的选择集的对象 - 再次,这个*可以*被字符串替换: 

```js(nocopy)
const selectionSet = `
{
  id
}
`
context.db.mutation.createLink({
  data: {
    url: "www.prisma.io"
    description: "Prisma turns your database into a GraphQL API"
  }
}, selectionSet)
```

这将对应于发送到API的以下突变: 

```graphql(nocopy)
mutation {
  createLink(data: {
    url: "www.prisma.io"
    description: "Prisma turns your database into a GraphQL API"
  }) {
    id
  }
}
```

因此,总而言之,Prisma绑定允许您调用与GraphQL架构中定义的操作相对应的函数. 这些函数与它们接收的参数和返回的值具有相同的操作名称和相同的结构. 

但是,你如何确保你的解析器实际上可以访问那些神奇且经常提到的`Prisma`绑定实例?

### 创造`Prisma`绑定实例

在做任何其他事情之前,请继续执行JavaScript开发人员最喜欢的事情: 为项目添加新的依赖项😑

<Instruction>

在项目的根目录中 (不在内部`database`) ,运行以下命令: 

```bash(path=".../hackernews-node")
yarn add prisma-binding
```

</Instruction>

凉!现在你可以附上了`Prisma`绑定实例到`context`这样你的解析器就可以访问它了. 

<Instruction>

在`index.js`,更新实例化`GraphQLServer`看起来如下 (请注意,您需要替换值`endpoint`使用您自己的Prisma端点!) : 

```js{4-12}(path=".../hackernews-node/src/index.js")
const server = new GraphQLServer({
  typeDefs: './src/schema.graphql',
  resolvers,
  context: req => ({
    ...req,
    db: new Prisma({
      typeDefs: 'src/generated/prisma.graphql',
      endpoint: 'https://eu1.prisma.sh/public-graytracker-771/hackernews-node/dev',
      secret: 'mysecret123',
      debug: true,
    }),
  }),
})
```

</Instruction>

所以,这就是诀窍. 你在实例化`Prisma`以下信息: 

-   `typeDefs`: 这指向Prisma数据库模式,该模式定义了Prisma的完整CRUD GraphQL API. 请注意,您实际上还没有这个文件 - 我们会告诉您如何获取它. 
-   `endpoint`: 这是Prisma API的终结点. 不要忘记在这里用您自己的Prisma服务的端点替换它!
-   `debug`: 设置`debug`国旗`true`意味着所有请求,由`Prisma`将Prisma API的绑定实例记录到控制台. 这是观察发送到Prisma的实际GraphQL查询和突变的便捷方式. 

<Instruction>

最后,为了完成这项工作,您需要导入`Prisma`成`index.js`. 在文件的顶部,添加以下import语句: 

```js(path=".../hackernews-node/src/index.js")
const { Prisma } = require('prisma-binding')
```

</Instruction>

你几乎完成!这个拼图只有一个部分,而且正在下载Prisma数据库模式`Prisma`构造函数调用. 

### 下载Prisma数据库架构

有多种方法可以访问GraphQL API的模式. 在本教程中. 你会用的[GraphQL CLI](https://github.com/graphql-cli/graphql-cli)与...结合[`graphql-config`](https://github.com/graphcool/graphql-config). 这也可以为您的一般工作流程带来更多改进. 

<Instruction>

首先,创建一个`.graphqlconfig`文件: 

```bash(path=".../hackernews-node")
touch .graphqlconfig.yml
```

</Instruction>

此文件是GraphQL CLI的主要信息源. 

<Instruction>

添加以下内容到`.graphqlconfig.yml`: 

```yml(path=".../hackernews-node/.graphqlconfig.yml")
projects:
  app:
    schemaPath: src/schema.graphql
    extensions:
      endpoints:
        default: http://localhost:4000
  database:
    schemaPath: src/generated/prisma.graphql
    extensions:
      prisma: database/prisma.yml
```

</Instruction>

那么,这里发生了什么?你定义了两个`projects`. 正如您可能猜到的,每个项目代表您的GraphQL API之一 - 应用程序层 (`graphql-yoga`) 和数据库层 (Prisma) . 

对于每个项目,您都指定了一个`schemaPath`. 这只是定义每个API的GraphQL架构的路径. 

为了`app`项目,你进一步指定一个`endpoint`这是GraphQL服务器启动时运行的URL. 

该`database`项目另一方面只指向了`prisma.yml`文件. 实际上,指向此文件还提供有关Prisma服务端点的信息,因为可以在那里找到组成端点所需的所有信息. 

您现在可以从此设置中获得两个主要好处: 

-   您可以与Playground中的两个GraphQL API并排进行交互. 
-   部署Prisma服务时`prisma deploy`,Prisma CLI将生成的Prisma数据库模式下载到指定位置. 

Prisma CLI还使用了提供的信息`.graphqlconfig.yml`. 因此,您现在可以运行了`prisma`来自根目录而不是来自的目录`database`目录. 

<Instruction>

更新`prisma.yml`包含一个部署钩子: 

```bash(path=".../hackernews-node/database/prisma.yml")
endpoint: `https://eu1.prisma.sh/public-graytracker-771/hackernews-node/dev
datamodel: datamodel.graphql

# Deploy hook
hooks:
  post-deploy:
    - graphql get-schema --project database
```

</Instruction>

当Prisma完成部署时,将调用部署挂钩. 在这种情况下,我们想要使用下载架构`get-schema`命令并指向`database`已配置的项目`.graphqlconfig.yml`. 

部署钩子调用一个`graphql`因此,CLI命令需要全局安装: 

<Instruction>

安装GraphQL CLI: 

```bash
yarn global add graphql-cli
```

</Instruction>

安装CLI工具后,您可以再次启动部署过程: 

<Instruction>

在项目的根目录中,运行`prisma deploy`将Prisma数据库模式下载到指定的位置`.graphqlconfig.yml`: 

```bash(path=".../hackernews-node")
prisma deploy
```

</Instruction>

观察命令的输出,你可以看到它这次打印以下行: 

```(nocopy)
Writing database schema to `src/generated/prisma.graphql`  1ms
```

瞧,还有Prisma数据库架构`src/generated/prisma.graphql`😮

好的,在您开始并测试服务器之前,最后一个小的改变!

<Instruction>

打开`src/schema.graphql`并删除`Link`类型. 

</Instruction>

恩,什么?为什么要这么做?在哪里`Link`现在定义来自于定义中使用的定义`feed`和`post`领域. 答: 你会导入它. 

<Instruction>

还在`src/schema.graphql`,将以下import语句添加到文件的顶部: 

```graphql(path=".../hackernews-node/src/schema.graphql")
# import Link from "./generated/prisma.graphql"
```

</Instruction>

此处使用的此导入语法不是官方GraphQL规范的一部分 ([然而!](https://github.com/graphql/graphql-wg/blob/master/notes/2018-02-01.md#present-graphql-import)) . 它来自于[`graphql-import`](https://github.com/graphcool/graphql-import)正在使用的包裹`graphql-yoga`解决任何依赖关系`.graphql`文件. 

请注意,在这种情况下,如果你离开它,它实际上不会有所作为`Link`按原样输入. 但是,仅定义它更方便`Link`类型*一旦*然后重用该定义. 否则,每当您更改时,您都必须更新多个定义`Link`类型. 

好的,就是这样!您现在可以立即启动服务器并立即测试API!

### 在同一个Playground中访问两个GraphQL API

现在让我们来看看如何利用这些信息`.graphqlconfig.yml`同时使用两个GraphQL API. 

跑过`graphql playground`命令一次打开两个API. 在此之前,您需要启动GraphQL服务器 (否则为Playground服务器) `app`项目不起作用) . 

<Instruction>

接下来,启动GraphQL服务器: 

```bash(path=".../hackernews-node")
node src/index.js
```

</Instruction>

<Instruction>

现在,打开一个新的终端选项卡 (或窗口) 并运行以下命令在同一GraphQL Playground中打开两个GraphQL API: 

```bash(path="hackernews-node/")
graphql playground
```

</Instruction>

> **注意**: 你也可以下载[桌面应用](https://github.com/graphcool/graphql-playground/releases)GraphQL Playground而不是在浏览器中使用它. 

以下是两个项目的Playground的样子: 

![](https://imgur.com/uEPCMs5.png)

使用左侧栏,您现在可以在不同的项目之间切换,并将请求发送到应用程序层或数据库层✌️️

好吧,就是这样*很多*配置,只有很少的编码!让我们改变它并实现更多功能. 
