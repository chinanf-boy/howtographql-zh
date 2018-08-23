---
title: Introduction
pageTitle: "Building a GraphQL Server with Node.js & Prisma Tutorial"
description: "Learn how to build a GraphQL server with graphql-yoga, Node.js & Prisma and best practices for authentication, filtering, pagination and subscriptions."
question: What is a GraphQL Playground?
answers: ["A GraphQL IDE to work with a GraphQL API", "A tool to generate GraphQL operations", "A REST client", "The successor of Postman"]
correctAnswer: 0
---
> **注意**: 可以在上找到本教程的最终项目[GitHub上](https://github.com/howtographql/graphql-js). 每当您在以下章节的过程中迷路时,您始终可以将其用作参考. 另请注意,每个代码块都使用文件名进行注释. 这些注释直接链接到GitHub上的相应文件,因此您可以清楚地看到放置代码的位置以及最终结果的样子. 

### 概观

GraphQL是后端技术的后起之秀. 它取代了REST作为API设计范例,并且正在成为公开服务器数据和功能的新标准. 

在本教程中,您将学习如何构建*惯用的*GraphQL服务器完全从头开始. 您将使用以下技术: 

-   [`graphql-yoga`](https://github.com/graphcool/graphql-yoga): 功能齐全的GraphQL服务器,专注于轻松设置,性能和出色的开发人员体验. 它建立在之上[表现](https://expressjs.com/),[`apollo-server`](https://github.com/apollographql/apollo-server),[`graphql-js`](https://github.com/graphql/graphql-js)和更多. 
-   [Prisma](https://www.prisma.io/): GraphQL数据库代理,可将您的数据库转换为GraphQL API. 此API为您的数据模型提供强大的实时CRUD操作. 
-   [`graphql-config`](https://github.com/graphcool/graphql-config)&[GraphQL CLI](https://github.com/graphql-cli/graphql-cli): 工具,以改善各种与GraphQL相关的worfklows. 
-   [GraphQL绑定](https://blog.graph.cool/reusing-composing-graphql-apis-with-graphql-bindings-80a4aa37cff5): 使用GraphQL API的便捷方式. 绑定为每个API操作生成专用的JavaScript函数. 
-   [GraphQL游乐场](https://github.com/graphcool/graphql-playground): "GraphQL IDE",允许通过向其发送查询和突变来交互式地探索GraphQL API的功能. 它有点类似于[邮差](https://www.getpostman.com/)它为REST API提供了类似的功能. 除其他外,GraphQL Playground ......
    -   ...自动生成所有可用API操作的综合文档. 
    -   ...提供了一个编辑器,您可以使用自动完成 (!) 和语法突出显示来编写查询,突变和订阅. 
    -   ...让您轻松共享API操作. 

### 会发生什么

本教程的目标是为a构建API[黑客新闻](https://news.ycombinator.com/)克隆. 以下是本教程中期望内容的快速概述. 

您将首先学习GraphQL服务器的工作原理,只需定义一个[*GraphQL架构*](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e)为服务器和写作相应的*解析器功能*. 一开始,这些解析器只能处理存储在内存中的数据 - 因此在服务器的运行时之外不会持久存储任何内容. 

因为没有人想要一个无法存储和保存数据的服务器,所以你要为它添加一个数据库层. 数据库层由. 提供支持[Prisma](https://www.prisma.io/)并将通过您的GraphQL服务器连接[Prisma绑定](https://github.com/graphcool/prisma-binding). 您可以将这些绑定视为"GraphQL ORM",帮助您正确解决传入的查询. 

连接数据库后,您将为API添加更多高级功能. 

您将首先实现注册/登录功能,使用户能够对API进行身份验证. 这还允许您检查用户对某些API操作的权限. 

本教程的下一部分是使用GraphQL订阅向您的API添加实时功能. 

最后,您将允许API的使用者通过向API添加过滤和分页功能来约束他们从API检索的项目列表. 

我们马上开始吧🚀
