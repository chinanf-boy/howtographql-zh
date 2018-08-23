---
title: A Simple Mutation
pageTitle: "Implementing Mutations in a GraphQL Server Tutorial"
description: "Learn best practices for implementing GraphQL mutations with graphql-js, Node.js & Prisma. Test your implementation in a GraphQL Playground."
question: What is the second argument that's passed into GraphQL resolvers used for?
answers: ["It carries the return value of the previous resolver execution level", "It carries the arguments for the incoming GraphQL operation", "It is an object that all resolvers can write to and read from", "It carries the AST of the incoming GraphQL operation"]
correctAnswer: 1
---
在本节中,您将学习如何向GraphQL API添加变异. 这种突变将允许*岗位*新的服务器链接. 

### 扩展架构定义

像以前一样,您需要首先将新操作添加到GraphQL架构定义中. 

<Instruction>

在`index.js`,扩展`typeDefs`字符串如下: 

```js{7-9}(path="../hackernews-node/src/index.js")
const typeDefs = `
type Query {
  info: String!
  feed: [Link!]!
}

type Mutation {
  post(url: String!, description: String!): Link!
}

type Link {
  id: ID!
  description: String!
  url: String!
}
`
```

</Instruction>

此时,模式定义已经变得非常大. 让我们稍微重构一下应用程序并将模式拉出到自己的文件中!

<Instruction>

在里面创建一个新文件`src`目录并调用它`schema.graphql`: 

```bash(path="../hackernews-node/src)
touch src/schema.graphql
```

</Instruction>

<Instruction>

接下来,将整个模式定义复制到新文件中: 

```graphql(path="../hackernews-node/src/schema.graphql)
type Query {
  info: String!
  feed: [Link!]!
}

type Mutation {
  post(url: String!, description: String!): Link!
}

type Link {
  id: ID!
  description: String!
  url: String!
}
```

</Instruction>

有了这个新文件,你就可以清理了`index.js`一点点. 

<Instruction>

首先,完全删除该定义`typeDefs`常量 - 不再需要它,因为模式定义现在存在于自己的文件中. 然后,更新方式如何`GraphQLServer`在文件的底部实例化: 

```js{2}(path="../hackernews-node/src/index.js)
const server = new GraphQLServer({
  typeDefs: './src/schema.graphql',
  resolvers,
})
```

</Instruction>

关于构造函数的一个方便的事情`GraphQLServer`就是它`typeDefs`可以直接提供为字符串 (如前所述) ,也可以通过引用包含模式定义的文件 (这就是您现在正在做的事情) . 

### 实现解析器功能

向API添加新功能的过程的下一步是为新字段实现解析器功能. 

<Instruction>

接下来,更新`resolvers`功能如下: 

```js{7,13-23}(path="../hackernews-node/src/index.js")
let links = [{
  id: 'link-0',
  url: 'www.howtographql.com',
  description: 'Fullstack tutorial for GraphQL'
}]
// 1
let idCount = links.length
const resolvers = {
  Query: {
    info: () => `This is the API of a Hackernews Clone`,
    feed: () => links,
  },
  Mutation: {
    // 2
    post: (root, args) => {
       const link = {
        id: `link-${idCount++}`,
        description: args.description,
        url: args.url,
      }
      links.push(link)
      return link
    }
  },
}
```

</Instruction>

首先,请注意你完全删除了`Link`解析器 (如前所述) . 它们不是必需的,因为GraphQL服务器推断出它们的外观. 

此外,以下是编号注释的内容: 

1.  您正在添加一个新的整数变量,它仅用作为新创建的唯一ID生成的方法`Link`元素. 
2.  执行`post`解析器首先创建一个新的`link`对象,然后将其添加到现有的`links`列表,最后返回新的`link`. 

现在是讨论传递给所有解析器函数的第二个参数的好时机: `args`. 有没有猜到它用于什么?

正确!它承载着*参数*对于操作 - 在这种情况下`url`和`description`的`Link`要被创造. 我们不需要它`feed`和`info`之前解析器,因为相应的根字段未在架构定义中指定任何参数. 

### 测试突变

继续并重新启动服务器,以便测试新的API操作. 以下是您可以通过Playground发送的样本变种: 

```graphql
mutation {
  post(
    url: "www.prisma.io"
    description: "Prisma turns your database into a GraphQL API"
  ) {
    id
  }
}
```

服务器响应如下所示: 

```json(nocopy)
{
  "data": {
    "post": {
      "id": "link-1"
    }
  }
}
```

随着你发送的每一个突变,`idCount`将增加并创建链接的以下ID`link-2`,`link-3`等等......

要验证您的突变是否有效,您可以发送`feed`从之前再次查询 - 它现在返回您使用突变创建的其他帖子: 

![](https://i.imgur.com/l5wOvFI.png)

但是,一旦您终止并重新启动服务器,您会注意到之前添加的链接现已消失,您需要再次添加它们. 这是因为仅存储链接*在记忆中*, 在里面`links`阵列. 在接下来的部分中,您将学习如何添加*数据库层*到GraphQL服务器,以便将数据持久化到服务器的运行时之外. 

### 行使

如果你想更多地练习GraphQL解析器,这对你来说是一个有趣的小挑战. 根据您当前的实现,扩展具有完整CRUD功能的GraphQL API`Link`类型. 特别是,实现具有以下定义的查询和突变: 

```graphql(nocopy)
type Query {
  # Fetch a single link by its `id`
  link(id: ID!): Link
}

type Mutation {
  # Update a link
  updateLink(id: ID!, url: String, description: String): Link

  # Delete a link
  deleteLink(id: ID!): Link
}
```
