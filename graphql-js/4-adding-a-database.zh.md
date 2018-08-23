---
title: Adding a Database
pageTitle: "Creating a Prisma Database Service Tutorial"
description: "Learn how to add a database to your GraphQL server. The database is powered by Prisma and connected to the server via GraphQL bindings."
question: Why is a second GraphQL API (defined by the application schema) needed in a GraphQL server architecture with Prisma?
answers: ["To increase performance of the server", "When using two APIs, the GraphQL server can be scaled better", "The Prisma API only is an interface to the database, but doesn't allow for any sort of application logic which is needed in most apps", "It is required by the GraphQL specification"]
correctAnswer: 2
---
在本节中,您将设置Prisma服务以及服务器要使用的已连接数据库. 

### 为何选择Prisma

到目前为止,您已经了解了GraphQL服务器如何工作的基本机制 - 非常简单对吧?这是GraphQL之美的一部分,它实际上只遵循一些非常简单的规则. 强类型模式和解析服务器内部查询的GraphQL引擎正在消除API开发中常见的主要难点. 

那么,构建GraphQL服务器的难度是什么呢?

好吧,在实际应用程序中,您可能会遇到许多场景,其中实现解析器会变得非常复杂. 特别是因为GraphQL查询可以嵌套多个级别,实现通常变得棘手并且很容易导致性能问题. 

大多数情况下,您还需要处理许多其他工作流程,例如身份验证,授权 (权限) ,分页,过滤,实时,与第三方服务或遗留系统集成......

通常,在实现解析器并连接到数据库时,您有两个选项 - 这两个选项都不是很吸引人: 

-   直接访问数据库 (通过编写SQL或使用其他NoSQL数据库API) 
-   用一个[ORM](https://en.wikipedia.org/wiki/Object-relational_mapping)它为您的数据库提供了一个抽象,并允许您直接从您的编程语言访问它

第一个选项是有问题的,因为在解析器中处理SQL很复杂,很快就会失控. 另一个问题是SQL查询通常以普通方式提交给数据库*字符串*. 字符串不遵循任何结构,它们只是原始的字符序列. 因此,您的工具将无法帮助您发现任何问题,或者在编辑器中提供自动完成等额外补贴. 编写SQL查询因此繁琐且容易出错. 

第二种选择是使用ORM,这可能看起来是一个很好的解决方案. 但是,这种方法通常也会缩短. ORM通常存在这样的问题: 他们正在为数据库访问实现相当简单的解决方案,当使用GraphQL时,由于查询的复杂性和可能出现的各种边缘情况,它们将无法工作. 

Prisma为您提供了一个解决这个问题的方法*GraphQL查询引擎*这是负责为您解决查询. 使用Prisma时,您正在实施您的解析器,这样他们就可以了*委托*对底层Prisma引擎的传入查询. 谢谢[Prisma绑定](https://github.com/graphcool/prisma-binding),查询委派是一个简单的过程,大多数解析器可以作为简单的单行实现. 

> **注意**: Prisma绑定基于模式拼接和模式委派的思想. 我们不打算在本教程中详细介绍这些技术. 如果您想了解更多相关信息,可以查看以下两篇文章: 
>
> -   [GraphQL Schema Stitching解释: 架构委派](https://blog.graph.cool/graphql-schema-stitching-explained-schema-delegation-4c6caf468405)
> -   [使用GraphQL绑定重用和组合GraphQL API](https://blog.graph.cool/reusing-composing-graphql-apis-with-graphql-bindings-80a4aa37cff5)

### 建筑

以下是使用Prisma构建GraphQL服务器时使用的体系结构概述: 

![](https://imgur.com/ik5P7RO.png)

了解这个与两个 (!) GraphQL API层相关的架构非常重要. 

#### 应用层

第一个GraphQL API是您在本教程前面部分中已经开始构建的API. 这是用于的GraphQL API**应用层**. 它定义了客户端应用程序要与之通信的API. 这是您实施的地方*商业逻辑*,常见的工作流程*认证*和*授权*或与...整合*第三方服务* (如果您想要实施付款流程,请使用Stripe) . 应用程序层的API由GraphQL架构定义`src/schema.graphql`- 因此,我们从现在开始将此模式称为**应用模式**. 

#### 数据库层

第二个GraphQL API是由Prisma提供并提供的**数据库层**. 这个基本上是一个基于GraphQL的数据库接口,可以让您免于自己编写SQL的复杂性. 那么,GraphQL API是什么样的?

Prisma API正在镜像数据库API,因此它允许您执行特定的CRUD操作*数据类型*. 什么数据类型?嗯,这取决于您 - 您使用熟悉的SDL定义这些数据类型. 您将了解它的工作原理. 

通常,这些数据类型代表*您的应用程序域的实体*. 例如,如果您正在构建汽车经销商软件,那么您可能会拥有诸如此类的数据类型`Car`,`CarDealer`,`Customer`等等......这些数据类型的整个集合称为您的*数据模型*.

Once your data model is defined in SDL, Prisma translates it into an according database schema and sets up the underlying database accordingly🥄 When you're then sending queries and mutations to the Prisma GraphQL API, it translates those into database operations and performs these operations for you🥄 Neat, right?

Previously you learned that all GraphQL APIs are backed by a GraphQL schema🥄 So, who is writing the schema for the Prisma GraphQL API🥄 The answer is that it is*automatically generated*based on the data model you provide🥄 By the way, this schema is called the**Prisma database schema**.

As an example, consider this simple data model with a single`User`type:

```graphql(nocopy)
type User {
  id: ID! @unique
  name: String!
}
```

> **Note**: Don't worry about the`@unique`directive yet, we'll talk about it soon.

Based on this data model, Prisma would generate a GraphQL schema looking like this:

```graphql(nocopy)
type Query {
  users(where: UserWhereInput, orderBy: UserOrderByInput, skip: Int, after: String, before: String, first: Int, last: Int): [User]!
  user(where: UserWhereUniqueInput!): User
}

type Mutation {
  createUser(data: UserCreateInput!): User!
  updateUser(data: UserUpdateInput!, where: UserWhereUniqueInput!): User
  deleteUser(where: UserWhereUniqueInput!): User
}

type Subscription {
  user(where: UserSubscriptionWhereInput): UserSubscriptionPayload
}
```

In fact, the actual schema is quite a bit bigger - for brevity we've only included the three root types and the simple CRUD operations here🥄 But the API also allows for a variety of other operations (such as batched updates and deletes)🥄 If you're curious, you can check out the entire schema[here](https://gist.github.com/gc-codesnippets/3f4178ad93c51d03195c92ce119d444c).

#### Why not just use the Prisma GraphQL API directly?

Prisma really only is an interface to a database🥄 If you consumed the Prisma API directly from your frontend or mobile applications, this would be similar as*directly accessing a database*.

In very, very rare cases, this might be an option - but the vast majority of applications do need additional logic that is not covered by CRUD operations (data validation and transformation, authentication, permissions, integration of 3rd-party services or any other sort of custom functionality...).

Another potential concern of directly exposing the Prisma API to your client applications is*security*🥄 GraphQL works in the way that everyone who has access to the endpoint of a GraphQL API can retrieve the*entire*GraphQL schema from it - this is called[introspection](http://graphql.org/learn/introspection/)🥄 If your clients were talking directly to Prisma, it would be simply a matter of checking the network requests to get access to the endpoint of the Prisma API and everyone would be able to see your entire database schema.

> **Note**: 目前有争议是否应该可以限制内省功能,但到目前为止,它似乎并不是GraphQL规范开发中的优先事项. 看到[这个](https://github.com/graphql/graphql-js/issues/113)GitHub问题了解更多信息. 

### 使用已连接的数据库创建Prisma服务

在本教程中,您将完全从头开始构建所有内容!对于Prisma数据库服务,您将从可能的最小设置开始. 

您需要做的第一件事是创建两个文件,您将把它们放入一个名为的新目录中`database`. 

<Instruction>

首先,创建`database`目录然后调用两个文件`prisma.yml`和`datamodel.graphql`在终端中运行以下命令: 

```bash(path=".../hackernews-node/")
mkdir database
touch database/prisma.yml
touch database/datamodel.graphql
```

</Instruction>

[`prisma.yml`](https://www.prisma.io/docs/reference/service-configuration/prisma.yml/overview-and-example-foatho8aip)是Prisma数据库服务的主要配置文件. `datamodel.graphql`另一方面,包含数据模型的定义,它将成为Prisma生成的GraphQL CRUD API的基础. 

到目前为止,您的Hacker News应用程序的数据模型只包含一种数据类型: `Link`. 实际上,你基本上可以复制现有的`Link`来自的定义`schema.graphql`成`datamodel.graphql`. 

<Instruction>

打开`datamodel.graphql`并添加以下代码: 

```graphql{2,3}(path=".../hackernews-node/database/datamodel.graphql")
type Link {
  id: ID! @unique
  createdAt: DateTime!
  description: String!
  url: String!
}
```

</Instruction>

与以前相比有两个主要差异`Link`版本来自`schema.graphql`. 

首先,你要添加`@unique`指令`id: ID!`领域. 该指令通常告诉Prisma你永远不会想要任何两个`Link`数据库中具有该字段相同值的元素. 事实上,`id: ID!`是Prisma数据模型中的一个特殊字段,因为Prisma将为具有此字段的类型自动生成全局唯一ID. 

其次,你要添加一个名为的新字段`createdAt: DateTime!`. 此字段也由Prisma管理,并且在API中是只读的. 它存储特定时间`Link`创建了. 请注意,Prisma提供了另一个类似的字段,称为`updatedAt: DateTime`- 这个存储时间`Link`最后更新. 

现在,让我们看看你需要做些什么`prisma.yml`. 

<Instruction>

添加以下内容到`prisma.yml`: 

```graphql(path=".../hackernews-node/database/prisma.yml")
# The HTTP endpoint for your Prisma API
endpoint: ''

# Points to the file that holds your data model
datamodel: datamodel.graphql

# You can only access the API when providing JWTs that are signed with this secret
secret: mysecret123
```

</Instruction>

要了解有关结构的更多信息`prisma.yml`,随便看看[文件](https://www.prisma.io/docs/reference/service-configuration/prisma.yml/yaml-structure-ufeshusai8). 

以下是您在该文件中看到的每个属性的快速说明: 

-   `endpoint`: Prisma API的HTTP端点. 实际上需要部署Prisma API. 我们部署时会进行调整. 
-   `datamodel`: 这只是指向*数据模型*这是Prisma CRUD API的基础. 
-   `secret`: 您希望保护您的Prisma服务,并要求对您的Prisma API进行身份验证. 这个*秘密用于签署JWT*这需要包括在内`Authorization`针对API发出的任何HTTP请求的标头. 阅读更多相关信息[这里](https://www.prisma.io/docs/reference/prisma-api/concepts-utee3eiquo#authentication). 

在部署服务之前,您需要安装用于管理Prisma服务的Prisma CLI. 

<Instruction>

要使用NPM全局安装Prisma CLI,请使用以下命令: 

```bash
npm install -g prisma
```

</Instruction>

好的,您终于可以部署Prisma服务和随附的数据库了!🙌

<Instruction>

导航到`database`目录并运行[`prisma deploy`](https://www.prisma.io/docs/reference/cli-command-reference/database-service/prisma-deploy-kee1iedaov): 

```bash(path=".../hackernews-node/")
cd database
prisma deploy
```

</Instruction>

该`prisma deploy`命令启动一个交互过程: 

<Instruction>

首先选择**演示服务器**从提供的选项. 浏览器打开时**注册Prisma Cloud**然后回到你的终端. 

</Instruction>

<Instruction>

然后你需要选择**地区**为您的演示服务器. 完成后,您只需按两次Enter键即可使用建议值**服务**和**阶段**. 

</Instruction>

> **注意**: Prisma是开源的. 它基于[搬运工人](http://docker.com/)这意味着您可以将其部署到您选择的任何云提供商 (例如Digital Ocean,AWS,Google Cloud,......) . 如果您不想处理DevOps和Docker的手动配置,您也可以使用[Prisma Cloud](https://blog.graph.cool/introducing-prisma-cloud-a-graphql-database-platform-ed591baa8737)轻松启动可以部署服务的专用群集. 看这个简短[视频](https://www.youtube.com/watch?v=jELE4KXJPn4)了解更多有关工作原理的信息. 

命令运行完毕后,CLI将输出Prisma GraphQL API的端点. 它看起来有点类似于: `https://eu1.prisma.sh/public-graytracker-771/hackernews-node/dev`. 

以下是URL的组成方式: 

-   `eu1.prisma.sh`: 群集的域
-   `public-graytracker-771`: 随机生成的服务ID
-   `hackernews-node`: 来自的服务名称`prisma.yml`
-   `dev`: 部署阶段`prisma.yml`

在将来的部署中 (例如,在对数据模型进行更改之后) ,将不再提示您在何处部署服务 -  CLI将从中读取端点URL`prisma.yml`. 

### 探索Prisma服务

要浏览Prisma数据库API,请打开CLI打印的URL. 

> **注意**: 如果您丢失了端点,则可以通过运行再次访问它`prisma info`在终端. 

不幸的是,如果您这样做,您将受到错误的欢迎: 

```json(nocopy)
{
  "errors": [
    {
      "message": " Your token is invalid. It might have expired or you might be using a token from a different project.",
      "code": 3015,
      "requestId": "api:api:cjfcbpal10t6w0b91idqif941"
    }
  ]
}
```

请记住我们如何说您的Prisma API受到保护`secret`从`prisma.yml`?嗯,这正是你现在收到错误的原因. Playground正在尝试从端点加载GraphQL架构,但其请求未经过身份验证. 让我们继续改变它. 

在 - 的里面`database`目录,运行以下命令以生成使用. 签名的身份验证令牌 (JWT) `secret`从`prisma.yml`: 

```bash(path=".../hackernews-node/database")
prisma token
```

然后复制CLI打印的令牌并使用它在Playground中配置HTTP标头. 你可以打开这个**HTTP HEADERS**Playground左下角的窗格 - 请注意您需要替换`__TOKEN__`占位符与打印的实际令牌: 

```json
{
  "Authorization": "Bearer __TOKEN__"
}
```

它看起来与此类似: 

```json(nocopy)
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkYXRhIjp7InNlcnZpY2UiOiJoYWNrZXJuZXdzLW5vZGVAZGV2Iiwicm9sZXMiOlsiYWRtaW4iXX0sImlhdCI6MTUyMjMxNjM2MCwiZXhwIjoxNTIyOTIxMTYwfQ.MUoHGvw61iIq45ZVInOoylcs6_q2ldfD_GjQOVBqEqY"
}
```

几秒钟后,Playground将加载架构,您可以将经过身份验证的请求发送到Prisma API. 打开文档以检查可用的API操作. 

![](https://imgur.com/CK1xXWq.png)

如果您愿意,可以发送以下变异和查询以创建新链接,然后检索链接列表: 

创建一个新的`Link`: 

```graphql
mutation {
  createLink(data: {
    url: "www.prisma.io"
    description: "Prisma turns your database into a GraphQL API"
  }) {
    id
  }
}
```

全部加载`Link`内容: 

```graphql
query {
  links {
    id
    url
    description
  }
}
```

![](https://imgur.com/jq6dOL7.png)
