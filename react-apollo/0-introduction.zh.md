---
title: Introduction
pageTitle: "Fullstack Tutorial with GraphQL, React & Apollo"
description: "Learn how to build a Hackernews clone with GraphQL, React & Apollo Client. Use create-react-app for the frontend and graphql-yoga & Prisma for the backend."
question: What's a major benefit of using a GraphQL client library?
answers: ["It makes it easy to use animations inside your app", "A GraphQL client is mainly used to improve security", "It saves you from writing infrastructure code for networking and caching", "GraphQL clients don't provide actual advantages but it's always good to use 3rd party libraries"]
correctAnswer: 2
videoId: ""
duration: 0
videoAuthor: ""
---
**注意: **可以在上找到本教程的最终项目[GitHub上](https://github.com/howtographql/react-apollo). 每当您在以下章节的过程中迷路时,您始终可以将其用作参考. 另请注意,每个代码块都使用文件名进行注释. 这些注释直接链接到GitHub上的相应文件,因此您可以清楚地看到放置代码的位置以及最终结果的样子. 

### 概观

在之前的教程中,您了解了GraphQL的主要概念和优点. 现在是时候让你的手弄脏并开始实际项目!

你将构建一个简单的克隆[Hackernews](https://news.ycombinator.com/). 以下是该应用程序将具有的功能列表: 

-   显示链接列表
-   搜索链接列表
-   用户可以进行身份​​验证
-   经过身份验证的用户可以创建新链接
-   经过身份验证的用户可以提供链接 (每个链接和用户一票) 
-   当其他用户提升链接或创建新链接时,实时更新

在此轨道中,您将使用以下技术构建应用程序: 

-   前端: 
    -   [应对](https://facebook.github.io/react/): 用于构建用户界面的前端框架
    -   [Apollo Client 2.1](https://github.com/apollographql/apollo-client): 生产就绪,缓存GraphQL客户端
-   后端: 
    -   [`graphql-yoga`](https://github.com/graphcool/graphql-yoga/): 功能齐全的GraphQL Server,专注于轻松设置,性能和出色的开发人员体验
    -   [Prisma](https://www.prismagraphql.com/): 开源GraphQL API层,可将您的数据库转换为GraphQL API

你将用它创建React项目[`create-react-app`](https://github.com/facebookincubator/create-react-app),一个流行的命令行工具,它为您提供了一个空白项目,其中已经设置了所有必需的构建配置. 

### 为什么选择GraphQL客户端?

在里面[客户端](/advanced/0-clients/)在GraphQL部分中,我们已经在更高层次上介绍了GraphQL客户端的职责,现在是时候更具体了. 

简而言之,您应该使用GraphQL客户端来处理与您正在构建的应用程序重复且不可知的任务. 例如,能够发送查询和突变,而不必担心较低级别的网络详细信息或维护本地缓存. 这是您在与GraphQL服务器交谈的任何前端应用程序中需要的功能 - 为什么当您可以使用其中一个出色的GraphQL客户端时自己构建它?

有一些GraphQL客户端库可用. 对于非常简单的用例 (例如编写脚本) ,[`graphql-request`](https://github.com/graphcool/graphql-request)可能已经足够满足您的需求. 但是,您可能正在编写一个更大的应用程序,您希望从缓存,乐观的UI更新和其他便利功能中受益. 在这些情况下,您可以选择[阿波罗客户](https://github.com/apollographql/apollo-client)和[中继](https://facebook.github.io/relay/). 

### 阿波罗vs接力

在前端开始使用GraphQL的人们最常见的问题是他们应该使用哪个GraphQL客户端. 我们将尝试提供一些提示,帮助您确定哪些客户是您下一个项目的正确客户!

[中继](https://facebook.github.io/relay/)Facebook是本土的GraphQL客户端,他们在2015年与GraphQL一起开源. 它结合了Facebook自2012年开始使用GraphQL以来收集的所有知识. 中继针对性能进行了大量优化,并尝试尽可能减少网络流量. 一个有趣的旁注是,Relay本身实际上是从一个开始的*路由*最终结合数据加载职责的框架. 因此,它演变成一个功能强大的数据管理解决方案,可以在JavaScript应用程序中用于与GraphQL API进行交互. 

继电器的性能优势是以显着的学习曲线为代价的. Relay是一个非常复杂的框架,理解它的所有部分确实需要一些时间来真正进入它. 随着1.0版本的发布,这方面的整体情况有所改善[接力现代](http://facebook.github.io/relay/docs/en/introduction-to-relay.html),但如果你想做点什么的话*刚开始*使用GraphQL,Relay可能还不是正确的选择. 

[阿波罗客户](https://github.com/apollographql/apollo-client)是一项社区驱动的工作,旨在构建易于理解,灵活且功能强大的GraphQL客户端. Apollo的目标是为人们用来构建Web和移动应用程序的每个主要开发平台构建一个库. 现在有一个JavaScript客户端,其中包含对流行框架的绑定[应对](https://github.com/apollographql/react-apollo),[角](https://github.com/apollographql/apollo-angular),[余烬](https://github.com/bgentry/ember-apollo-client)要么[Vue公司](https://github.com/Akryum/vue-apollo)以及早期版本[iOS版](https://github.com/apollographql/apollo-ios)和[Android的](https://github.com/apollographql/apollo-android)客户端. Apollo已经投入生产,并具有缓存,乐观UI,订阅支持等便利功能. 
