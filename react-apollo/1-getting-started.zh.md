---
title: Getting Started
pageTitle: "Getting Started with GraphQL, React & Apollo Tutorial"
description: Start building a Hackernews clone. Create the frontend with create-react-app and the backend with Prisma.
question: Why are there two GraphQL API layers in a backend architecture with Prisma?
answers: ["To increase robustness and stability of the GraphQL server (if one layer fails, the server is backed by the second one).", "To increase performance of the GraphQL server (requests are accelerated  by going through multiple layers).", "Prisma provides the database layer which offers CRUD operations. The second layer is the application layer for business logic and common workflows (like authentication).", "Having two GraphQL layers is a hard requirement by the GraphQL specification."]
correctAnswer: 2
draft: false
videoId: ""
duration: 0		
videoAuthor: ""
---
由于这是一个前端跟踪,您不会花时间实现后端. 相反,你将使用来自的服务器[节点教程](https://www.howtographql.com/graphql-js/0-introduction). 

创建React应用程序后,您将获取后端所需的代码. 

> **注意**: 可以在上找到本教程的最终项目[GitHub上](https://github.com/howtographql/react-apollo). 每当您在以下章节的过程中迷路时,您始终可以将其用作参考. 另请注意,每个代码块都使用文件名进行注释. 这些注释直接链接到GitHub上的相应文件,因此您可以清楚地看到放置代码的位置以及最终结果的样子. 

### 前端

#### 创建应用程序

首先,您将创建React项目!如开头所述,您将使用`create-react-app`为了那个原因. 

<Instruction>

如果您还没有,则需要安装`create-react-app`使用纱线: 

```bash
yarn global add create-react-app
```

</Instruction>

> **注意**: 本教程使用[纱](https://yarnpkg.com/)用于依赖管理. 查找有关如何安装它的说明[这里](https://yarnpkg.com/en/docs/install). 如果你喜欢使用`npm`,你可以运行等效的命令. 

<Instruction>

接下来,您可以使用它来引导您的React应用程序: 

```bash
create-react-app hackernews-react-apollo
```

</Instruction>

这将创建一个名为的新目录`hackernews-react-apollo`具有所有基本配置设置. 

通过导航到目录并启动应用程序,确保一切正常: 

```bash
cd hackernews-react-apollo
yarn start
```

这将打开浏览器并导航到`http://localhost:3000`应用程序正在运行的位置. 如果一切顺利,您将看到以下内容: 

![](http://imgur.com/Yujwwi6.png)

<Instruction>

为了改进项目结构,继续创建两个目录,两个目录都位于`src`夹. 第一个被称为`components`并将保留我们所有的React组件. 拨打第二个`styles`,那个是你将使用的所有CSS文件. 

`App.js`是一个组件,所以将其移入`components`. `App.css`和`index.css`包含样式,所以将它们移入`styles`. 您还需要更改对这些文件的引用`index.js`因此: 

</Instruction>

```js{4}(path=".../hackernews-react-apollo/src/index.js")
import React from 'react'
import ReactDOM from 'react-dom'
import './styles/index.css'
import App from './components/App'
```

您的项目结构现在应如下所示: 

```bash(nocopy)
.
├── README.md
├── node_modules
├── package.json
├── public
│   ├── favicon.ico
│   ├── index.html
│   └── manifest.json
├── src
│   ├── App.test.js
│   ├── components
│   │   └── App.js
│   ├── index.js
│   ├── logo.svg
│   ├── registerServiceWorker.js
│   └── styles
│       ├── App.css
│       └── index.css
└── yarn.lock
```

#### 准备造型

本教程是关于GraphQL的概念以及如何在React应用程序中使用它,因此我们希望在样式上花费最少的时间. 要减少此项目中CSS的使用,您将使用[超光速粒子](http://tachyons.io/)提供了许多CSS类的库. 

<Instruction>

打开`public/index.html`并添加第三个`link`标记在两个现有的Tachyons中的标记: 

```html{3}(path=".../hackernews-react-apollo/public/index.html")
<link rel="manifest" href="%PUBLIC_URL%/manifest.json">
<link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico">
<link rel="stylesheet" href="https://unpkg.com/tachyons@4.2.1/css/tachyons.min.css"/>
```

</Instruction>

由于我们仍然希望在这里和那里有更多的自定义样式,我们还为您准备了一些您需要包含在项目中的样式. 

<Instruction>

打开`index.css`并用以下内容替换其内容: 

```css(path=".../hackernews-react-apollo/src/styles/index.css")
body {
  margin: 0;
  padding: 0;
  font-family: Verdana, Geneva, sans-serif;
}

input {
  max-width: 500px;
}

.gray {
  color: #828282;
}

.orange {
  background-color: #ff6600;
}

.background-gray {
  background-color: rgb(246,246,239);
}

.f11 {
  font-size: 11px;
}

.w85 {
  width: 85%;
}

.button {
  font-family: monospace;
  font-size: 10pt;
  color: black;
  background-color: buttonface;
  text-align: center;
  padding: 2px 6px 3px;
  border-width: 2px;
  border-style: outset;
  border-color: buttonface;
  cursor: pointer;
  max-width: 250px;
}
```

</Instruction>

#### 安装Apollo客户端

<Instruction>

接下来,您需要提供Apollo Client (及其React绑定) 的功能,它包含在几个包中: 

```bash(path=".../hackernews-react-apollo")
yarn add apollo-boost react-apollo graphql
```

</Instruction>

以下是您刚刚安装的软件包的概述: 

-   [`apollo-boost`](https://github.com/apollographql/apollo-client/tree/master/packages/apollo-boost)通过捆绑使用Apollo Client时需要的几个软件包,可以提供一些便利: 
    -   `apollo-client`: 所有魔法发生的地方
    -   `apollo-cache-inmemory`: 我们推荐的缓存
    -   `apollo-link-http`: 用于远程数据获取的Apollo链接
    -   `apollo-link-error`: 用于错误处理的Apollo链接
    -   `apollo-link-state`: 用于当地国家管理的Apollo Link
    -   `graphql-tag`: 出口`gql`用于查询和突变的功能
-   [`react-apollo`](https://github.com/apollographql/react-apollo)包含将Apollo Client与React一起使用的绑定. 
-   [`graphql`](https://github.com/graphql/graphql-js)包含Facebook的GraphQL参考实现 -  Apollo Client也使用它的一些功能. 

就是这样,你准备写一些代码了!🚀

#### 配置`ApolloClient`

Apollo抽象出所有较低级别的网络逻辑,并为GraphQL服务器提供了一个很好的接口. 与使用REST API相比,您不必再处理构建自己的HTTP请求 - 而是可以简单地编写查询和突变并使用`ApolloClient`实例. 

使用Apollo时你要做的第一件事就是配置你的`ApolloClient`实例. 它需要知道*端点*您的GraphQL API,以便它可以处理网络连接. 

<Instruction>

打开`src/index.js`并用以下内容替换内容: 

```js{6-9,11-13,15-18,21-23}(path=".../hackernews-react-apollo/src/index.js")
import React from 'react'
import ReactDOM from 'react-dom'
import './styles/index.css'
import App from './components/App'
import registerServiceWorker from './registerServiceWorker'
import { ApolloProvider } from 'react-apollo'
import { ApolloClient } from 'apollo-client'
import { createHttpLink } from 'apollo-link-http'
import { InMemoryCache } from 'apollo-cache-inmemory'

const httpLink = createHttpLink({
  uri: 'http://localhost:4000'
})

const client = new ApolloClient({
  link: httpLink,
  cache: new InMemoryCache()
})

ReactDOM.render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>,
  document.getElementById('root')
)
registerServiceWorker()
```

</Instruction>

> 注意: 由...生成的项目`create-react-app`对字符串使用分号和双引号. 您要添加的所有代码都将使用**没有分号**而且大多数**单引号**. 您也可以自由删除任何现有的分号,并用单引号replace替换double

让我们试着了解该代码片段中发生了什么: 

1.  您正在从已安装的软件包导入所需的依赖项. 
2.  在这里你创建了`httpLink`这将连接你的`ApolloClient`使用GraphQL API的实例,您的GraphQL服务器将在其上运行`http://localhost:4000`. 
3.  现在你实例化了`ApolloClient`通过`httpLink`和一个新的实例`InMemoryCache`. 
4.  最后,渲染React应用程序的根组件. 该`App`被高阶组件包裹`ApolloProvider`通过了`client`作为道具. 

就是这样,你已准备好开始将一些数据加载到你的应用程序中!😎

### 后端

#### 下载服务器代码

如上所述,对于本教程中的后端,您只需使用最终项目即可[节点教程](https://www.howtographql.com/graphql-js/0-introduction). 

<Instruction>

在终端中,导航到`hackernews-react-apollo`目录并运行以下命令: 

```bash(path=".../hackernews-react-apollo")
curl https://codeload.github.com/howtographql/react-apollo/tar.gz/starter | tar -xz --strip=1 react-apollo-starter/server
```

</Instruction>

> **注意**: 如果您使用的是Windows,则可能需要安装[Git CLI](https://git-scm.com/)避免使用命令等潜在问题`curl`. 

您现在有一个名为的新目录`server`在您的项目中包含您后端所需的所有代码. 

在我们启动服务器之前,让我们快速了解主要组件: 

-   `database`: 此目录包含与Prisma数据库服务相关的所有文件. 
    -   `prisma.yml`是服务的根配置文件. 
    -   `datamodel.graphql`在GraphQL中定义您的数据模型[模式定义语言](https://blog.graph.cool/graphql-sdl-schema-definition-language-6755bcb9ce51) (SDL) . 数据模型是Prisma生成的GraphQL API的基础,它为数据模型中的类型提供了强大的CRUD操作. 
-   `src`: 此目录包含GraphQL服务器的源文件. 
    -   `schema.graphql`包含你的**应用模式**. 应用程序模式定义了您可以从前端发送的GraphQL操作. 我们稍后会仔细研究一下这个文件. 
    -   `generated/prisma.graphql`包含自动生成的**Prisma数据库架构**. Prisma模式为数据模型中的类型定义了强大的CRUD操作. 请注意,您永远不应手动编辑此文件,因为它会在数据模型更改时自动更新. 
    -   `resolvers`包含[*解析器功能*](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e#1880)用于应用程序模式中定义的操作. 
    -   `index.js`是GraphQL服务器的入口点. 

从提到的文件中,只定义了应用程序模式`server/src/schema.graphql`与您作为前端开发人员相关. 该文件包含[GraphQL架构](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e)它定义了您可以从前端应用程序发送的所有操作 (查询,突变和订阅) . 

这是它的样子: 

```graphql(path=".../hackernews-react-apollo/server/src/schema.graphql"&nocopy)
# import Link, Vote, LinkSubscriptionPayload, VoteSubscriptionPayload from "./generated/prisma.graphql"

type Query {
  feed(filter: String, skip: Int, first: Int, orderBy: LinkOrderByInput): Feed!
}

type Feed {
  links: [Link!]!
  count: Int!
}

type Mutation {
  post(url: String!, description: String!): Link!
  signup(email: String!, password: String!, name: String!): AuthPayload
  login(email: String!, password: String!): AuthPayload
  vote(linkId: ID!): Vote
}

type AuthPayload {
  token: String
  user: User
}

type User {
  id: ID!
  name: String!
  email: String!
}

type Subscription {
  newLink: LinkSubscriptionPayload
  newVote: VoteSubscriptionPayload
}
```

此架构允许以下操作: 

-   查询: 
    -   `feed`: 从后端检索所有链接,请注意此查询还允许过滤,排序和分页参数
-   突变: 
    -   `post`: 允许经过身份验证的用户创建新链接
    -   `signup`: 为新用户创建帐户
    -   `login`: 登录现有用户
    -   `vote`: 允许经过身份验证的用户投票选择现有链接
-   订阅: 
    -   `newLink`: 创建新链接时接收实时更新
    -   `newVote`: 提交投票时接收实时更新

例如,您可以发送以下内容`feed`查询从服务器检索前10个链接: 

```graphql(nocopy)
{
  feed(skip: 0, first: 10) {
    links {
      description
      url
      postedBy {
        name
      }
    }
  }
}
```

或者`signup`突变创建一个新用户: 

```graphql(nocopy)
mutation {
  signup(
    name: "Sarah",
    email: "sarah@graph.cool",
    password: "graphql"
  ) {
    token
    user {
      id
    }
  }
}
```

#### 部署Prisma数据库服务

在启动服务器并开始向其发送查询和突变之前,还有一件事要做. 需要部署Prisma数据库服务,以便服务器可以访问它. 

要部署服务,您只需安装服务器的依赖项并调用`prisma deploy`里面的命令`server`目录. 

<Instruction>

在终端中,导航到`server`目录并执行以下命令: 

```sh(path=".../hackernews-react-apollo/server")
yarn install
yarn prisma deploy
```

</Instruction>

请注意,您也可以省略`yarn prisma`如果你有上述命令`prisma`CLI全局安装在您的计算机上 (您可以使用它`yarn global add prisma`) . 在这种情况下,您可以简单地运行`prisma deploy`. 

<Instruction>

当系统提示您要部署服务的位置 (即哪个群集) 时,请选择任何开发群集,例如: `public-us1`要么`public-eu1`.  (如果安装了Docker,也可以在本地部署. ) 

</Instruction>

<Instruction>

从输出`deploy`命令,复制`HTTP`端点并将其粘贴到`server/src/index.js`,替换当前占位符`__PRISMA_ENDPOINT__`: 

```js(path=".../hackernews-react-apollo/server/src/index.js")
const server = new GraphQLServer({
  typeDefs: './src/schema.graphql',
  resolvers,
  context: req => ({
    ...req,
    db: new Prisma({
      typeDefs: 'src/generated/prisma.graphql',
      endpoint: '__PRISMA_ENDPOINT__',
      secret: 'mysecret123',
    }),
  }),
})
```

</Instruction>

> **注意**: 如果您丢失了端点,则可以通过运行再次访问它`yarn prisma info`. 

#### 探索服务器

有了适当的Prisma端点,您现在可以探索服务器了!

<Instruction>

导航到`server`目录并运行以下命令以启动服务器: 

```bash(path=".../hackernews-react-apollo/server")
yarn dev
```

</Instruction>

该`yarn dev`执行`dev`脚本定义于`package.json`. 该脚本首先启动服务器 (然后运行) `http://localhost:4000`) 然后打开一个[GraphQL游乐场](https://github.com/graphcool/graphql-playground)供您探索和使用API​​. 请注意,如果您只想在不打开Playground的情况下启动服务器,则可以通过运行来执行此操作`yarn start`. 

![](https://imgur.com/k2I7NJn.png)

> Playground是一个"GraphQL IDE",提供了一个交互式环境,允许向GraphQL API发送查询,变异和订阅. 它类似于一个工具[邮差](https://www.getpostman.com)您可以通过使用REST API了解它,但还有许多其他好处. 

关于Playground的首要注意事项是它有GraphQL API的内置文档. 此文档是基于GraphQL架构生成的,可以通过单击绿色来打开**SCHEMA**- 游乐场右边缘的按钮. 因此,它显示了您在上面的应用程序模式中看到的相同信息: 

![](https://imgur.com/8xK81qt.png)

关于Playground的另一个重要注意事项是它实际上允许您与两个 (!) GraphQL API进行交互. 

第一个是由你的定义**应用模式**,这是你的React应用程序将与之交互的那个. 它可以通过选择打开**默认**游乐场里**应用**项目在左侧菜单中. 

还有Prisma数据库API,它提供对存储在数据库中的数据的完全访问. 这个你可以通过选择打开**开发**游乐场里**数据库**项目在左侧菜单中. 此API由. 定义**Prisma架构** (在`server/src/generated/prisma.graphql`) . 

> **注意**: Playground从中接收有关两个GraphQL API的信息`.graphqlconfig.yml`列出这两个项目的文件**应用**和**数据库**. 这就是为什么它可以为您提供访问这两者的原因. 

为什么你需要两个GraphQL API?答案非常简单,您实际上可以将两个API视为两个*层*. 

应用程序模式定义了第一个层,也称为*应用层*. 它定义了客户端应用程序将能够发送到API的操作. 这包括业务逻辑以及其他常见功能和工作流程 (例如注册和登录) . 

第二层是*数据库层*由Prisma架构定义. 它提供了强大的CRUD操作,允许您执行*任何*对数据库中的数据进行一种操作. 

以下是该应用程序架构的概述: 

![](https://imgur.com/M8cbht4.png)

> **注意**: 当然可以*只要*使用前端的数据库API. 但是,在大多数实际应用程序中,您至少需要一些API无法提供的业务逻辑. 如果您的应用程序真的只需要执行CRUD操作并访问数据库,那么直接针对Prisma数据库API运行就完全没问题了. 

Playground的左侧窗格是*编辑*您可以用来编写查询,突变和订阅. 单击中间的播放按钮后,将发送您的请求,服务器的响应将显示在*结果*窗格在右边. 

<Instruction>

将以下两个突变复制到*编辑*窗格 - 确保拥有**默认**游乐场**应用**在左侧菜单中选择的项目: 

```graphql
mutation CreatePrismaLink {
  post(
    description: "Prisma turns your database into a GraphQL API 😎",
    url: "https://www.prismagraphql.com"
  ) {
    id
  }
}

mutation CreateApolloLink {
  post(
    description: "The best GraphQL client for React",
    url: "https://www.apollographql.com/docs/react/"
  ) {
    id
  }
}
```

</Instruction>

由于您一次向编辑器添加两个突变,因此需要进行突变*操作名称*. 在你的情况下,这些是`CreatePrismaLink`和`CreateApolloLink`. 

<Instruction>

点击**玩**- 在两个窗格中间的按钮,从下拉列表中选择每个突变一次. 

</Instruction>

![](https://imgur.com/2GViJwb.png)

这创造了两个新的`Link`数据库中的记录. 您可以通过在已打开的Playground中发送以下查询来验证突变是否真正有效: 

```graphql
{
  feed {
    links {
      id
      description
      url
    }
  }
}
```

> **注意**: 你也可以发送`feed`查询中**默认**游乐场里**应用**部分. 

如果一切顺利,查询将返回以下数据 (`id`因为它们是由Prisma生成的并且在全球范围内是独一无二的,所以你的情况当然会有所不同) : 

```json(nocopy)
{
  "data": {
    "feed": {
      "links": [
        {
          "id": "cjcnfwjeif1rx012483nh6utk",
          "description": "The best GraphQL client",
          "url": "https://www.apollographql.com/docs/react/"
        },
        {
          "id": "cjcnfznzff1w601247iili50x",
          "description": "Prisma turns your database into a GraphQL API 😎",
          "url": "https://www.prismagraphql.com"
        }
      ]
    }
  }
}
```

太棒了,你的服务器工作!👏
