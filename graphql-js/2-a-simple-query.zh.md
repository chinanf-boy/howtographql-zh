---
title: A Simple Query
pageTitle: "Resolving Queries with a GraphQL Server Tutorial"
description: "Learn how to define a GraphQL schema, implement query resolvers with Node.js and test your queries in a GraphQL Playground."
question: How are GraphQL queries resolved?
answers: ["With schema-driven development", "By invoking all available resolver functions", "By invoking the resolver function of the root field", "By invoking the resolver functions for the fields contained in the query"]
correctAnswer: 3
---
在本节中,您将实现第一个为黑客新闻克隆提供功能的API操作: 查询提要*链接*由其他用户发布的. 

### 扩展架构定义

让我们从实施一个开始`feed`查询,它允许您检索列表`Link`元素. 通常,在向API添加新功能时,每次进程看起来都非常相似: 

1.  使用new扩展GraphQL架构定义*根田* (和新的*数据类型*, 如果需要的话) 
2.  实施相应的*解析器功能*添加的字段

这个过程也被称为*架构驱动*要么*架构优先开发*. 

所以,让我们继续并解决第一步,扩展GraphQL架构定义. 

<Instruction>

在`index.js`,更新`typeDefs`常数如下: 

```js{4,7-11}(path="../hackernews-node/src/index.js")
const typeDefs = `
type Query {
  info: String!
  feed: [Link!]!
}

type Link {
  id: ID!
  description: String!
  url: String!
}
`
```

</Instruction>

很简单. 你正在定义一个新的`Link`表示可以发布到Hacker News的链接的类型. 每`Link`有一个`id`, 一个`description`和`url`. 然后,您将另一个根字段添加到`Query`允许您检索列表的类型`Link`元素. 这个清单保证永远不会`null` (如果有的话,它将是空的) 并且从不包含任何元素`null`- 这就是两个感叹号的用途. 

### 实现解析器功能

下一步是为the实现解析器功能`feed`查询. 事实上,我们还没有提到的一件事是,不仅如此*根域*,但实际上*所有*GraphQL架构中的类型字段具有解析器函数. 所以,你将添加解析器`id`,`description`和`url`的领域`Link`类型也是. 

<Instruction>

在`index.js`,添加一个包含虚拟数据的新列表并更新`resolvers`看起来如下: 

```js{2-6,12,15-19}(path="../hackernews-node/src/index.js")
// 1
let links = [{
  id: 'link-0',
  url: 'www.howtographql.com',
  description: 'Fullstack tutorial for GraphQL'
}]

const resolvers = {
  Query: {
    info: () => `This is the API of a Hackernews Clone`,
    // 2
    feed: () => links,
  },
  // 3
  Link: {
    id: (root) => root.id,
    description: (root) => root.description,
    url: (root) => root.url,
  }
}
```

</Instruction>

让我们再次浏览编号的评论: 

1.  该`links`变量用于在运行时存储链接. 目前,一切都只存储*在记忆中*而不是坚持在数据库中. 
2.  你正在为它添加一个新的解析器`feed`根田. 请注意,解析器始终必须以架构定义中的相应字段命名. 
3.  最后,你要为其上的字段添加三个解析器`Link`从架构定义中键入. 我们稍后会讨论一下`root`参数是在这里传递给解析器的. 

继续并通过重新启动服务器来测试实现 (首次使用**CTRL + C**如果服务器仍在运行,则停止服务器,然后执行`node src/index.js`再次) 并导航到`http://localhost:4000`在您的浏览器中. 如果您展开Playground的文档,您会注意到另一个调用的查询`feed`现在可用: 

![](https://imgur.com/0EQ5P9p.png)

通过发送以下查询来尝试: 

```graphql
query {
  feed {
    id
    url
    description
  }
}
```

很棒,服务器会使用您定义的数据进行响应`links`: 

```json(nocopy)
{
  "data": {
    "feed": [
      {
        "id": "link-0",
        "url": "www.howtographql.com",
        "description": "Fullstack tutorial for GraphQL"
      }
    ]
  }
}
```

通过从选择集中删除任何字段并观察服务器发送的响应,随意使用查询. 

### 查询解析过程

现在让我们快速讨论一下GraphQL服务器如何实际解析传入的查询. 正如您已经看到的,GraphQL查询包含许多*领域*它们的源代码在GraphQL架构的类型定义中. 

再次考虑上面的查询: 

```graphql(nocopy)
query {
  feed {
    id
    url
    description
  }
}
```

查询中指定的所有四个字段,`feed`,`id`,`url`和`description`也可以在架构定义中找到. 现在,你也学到了这一点*一切*模式定义中的字段由一个解析器函数支持,该函数负责返回精确该字段的数据. 

你能想象现在的查询解析过程是什么样的吗?实际上,GraphQL服务器必须执行的所有操作都是为查询中包含的字段调用所有解析程序函数,然后根据查询的形状打包响应. 因此,查询解析仅仅成为协调解析器函数调用的过程!

现在实施中仍然有点奇怪的一件事是解决方案`Link`所有人似乎都遵循一个非常简单和琐碎的模式: 

```js(nocopy)
Link: {
  id: (root) => root.id,
  description: (root) => root.description,
  url: (root) => root.url,
}
```

首先,重要的是要注意每个GraphQL解析器函数实际接收*四*输入参数. 由于现在我们的方案中不需要其余三个,我们只是省略它们. 别担心,你很快就会了解它们. 

第一个参数,通常称为`root` (要么`parent`) 是前一个结果*解析程序执行级别*. 但是,这是什么意思?🤔

好吧,正如您已经看到的,GraphQL查询可以*嵌套*. 每个嵌套级别 (即嵌套的花括号) 对应于一个解析器执行级别. 因此,上述查询具有两个这样的执行级别. 

在第一级,它调用`feed`解析并返回存储的全部数据`links`. 对于第二个执行级别,GraphQL服务器足够聪明,可以调用解析器`Link`类型 (因为模式,它知道`feed`返回一个列表`Link`在前一个解析程序级别返回的列表中的每个元素的元素) . 因此,在三者中的每一个`Link`解析器,传入`root`object是里面的元素`links`名单. 

> **注意**: 要了解更多相关信息,请查看[这个](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e#9d03)文章. 

无论如何,因为执行了`Link`解析器是微不足道的,你实际上可以省略它们,服务器将以与之前相同的方式工作
