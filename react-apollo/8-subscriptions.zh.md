---
title: Realtime Updates with GraphQL Subscriptions
pageTitle: "Realtime with GraphQL Subscriptions, React & Apollo Tutorial"
description: "Learn how to implement realtime functionality using GraphQL subscriptions with Apollo Client & React. The websockets will be handled by subscriptions-transport-ws."
question: "What transport does Apollo use to implement subscriptions?"
answers: ["WebSockets", "TCP", "UDP", "HTTP 2"]
correctAnswer: 0
videoId: ""
duration: 0		
videoAuthor: ""
---
本节的主要内容是使用GraphQL订阅将实时功能引入应用程序. 

### 什么是GraphQL订阅?

订阅是GraphQL功能,允许服务器在特定时间向其客户端发送数据*事件*发生. 订阅通常用[的WebSockets](https://en.wikipedia.org/wiki/WebSocket),服务器与客户端保持稳定连接. 这意味着在使用订阅时,你就是打破了*请求 - 响应周期*用于以前与API的所有交互. 客户端现在通过指定它感兴趣的事件来启动与服务器的稳定连接. 每次发生此特定事件时,服务器使用该连接将预期数据推送到客户端. 与阿波罗订阅

### 使用Apollo时,您需要配置您的

有关订阅端点的信息. `ApolloClient`这是通过添加另一个来完成的到阿波罗中间件链. `ApolloLink`这一次,就是这样来自`WebSocketLink`包. [`apollo-link-ws`](https://github.com/apollographql/apollo-link/tree/master/packages/apollo-link-ws)首先将此依赖项添加到您的应用中. 

打开终端并导航到项目的根目录. 

<Instruction>

然后执行以下命令: 

```bash(path=".../hackernews-react-apollo")
yarn add apollo-link-ws
```

> **注意**: 要使apollo-link-w工作,您还需要安装subscriptions-transport-ws

    yarn add subscriptions-transport-ws

</Instruction>

接下来,确保你的`ApolloClient`instance了解订阅服务器. 

<Instruction>

打开`index.js`并将以下导入添加到文件的顶部: 

```js(path=".../hackernews-react-apollo/src/index.js")
import { split } from 'apollo-link'
import { WebSocketLink } from 'apollo-link-ws'
import { getMainDefinition } from 'apollo-utilities'
```

</Instruction>

请注意,您现在也在导入`split`功能来自'apollo-link'. 

<Instruction>

现在创建一个新的`WebSocketLink`表示WebSocket连接. 使用`split`正确"路由"请求并更新构造函数调用`ApolloClient`像这样: 

```js(path=".../hackernews-react-apollo/src/index.js")
const wsLink = new WebSocketLink({
  uri: `ws://localhost:4000`,
  options: {
    reconnect: true,
    connectionParams: {
      authToken: localStorage.getItem(AUTH_TOKEN),
    }
  }
})

const link = split(
  ({ query }) => {
    const { kind, operation } = getMainDefinition(query)
    return kind === 'OperationDefinition' && operation === 'subscription'
  },
  wsLink,
  authLink.concat(httpLink)
)

const client = new ApolloClient({
  link,
  cache: new InMemoryCache()
})
```

</Instruction>

你是在实例化一个`WebSocketLink`知道订阅端点. 在这种情况下,订阅端点类似于HTTP端点,除了它使用`ws`代替`http`协议. 请注意,您还要验证与用户的websocket连接`token`你从中检索`localStorage`. 

[`split`](https://github.com/apollographql/apollo-link/blob/98eeb1deb0363384f291822b6c18cdc2c97e5bdb/packages/apollo-link/src/link.ts#L33)用于将请求"路由"到特定的中间件链接. 它需要三个参数,第一个是a`test`返回布尔值的函数. 剩下的两个参数也是类型`ApolloLink`. 如果`test`回报`true`,请求将被转发到作为第二个参数传递的链接. 如果`false`,到第三个. 

在你的情况下,`test`函数正在检查请求的操作是否为a*订阅*. 如果是这种情况,它将转发给`wsLink`否则 (如果是的话) *询问*要么*突变*) ,`authLink.concat(httpLink)`会照顾它: 

![](https://cdn-images-1.medium.com/max/720/1*KwnMO21k0d3UbyKWnlbeJg.png)
*照片取自[Apollo Link: 模块化GraphQL网络堆栈](https://dev-blog.apollodata.com/apollo-link-the-modular-graphql-network-stack-3b6d5fcf9244)通过[埃文斯豪瑟](https://twitter.com/EvansHauser)*

### 订阅新链接

要使应用程序在创建新链接时实时更新,您需要订阅正在发生的事件`Link`类型. 使用Prisma时,通常可以订阅三种事件​​: 

-   一个新的`Link`是*创建*
-   现有的`Link`是*更新*
-   现有的`Link`是*删除*

您将在中实现订阅`LinkList`组件,因为那是所有链接的呈现. 

<Instruction>

打开`LinkList.js`并更新当前组件如下: 

```js{11-13,18,22}(path=".../hackernews-react-apollo/src/components/LinkList.js")
class LinkList extends Component {
  _updateCacheAfterVote = (store, createVote, linkId) => {
    const data = store.readQuery({ query: FEED_QUERY })
  
    const votedLink = data.feed.links.find(link => link.id === linkId)
    votedLink.votes = createVote.link.votes
  
    store.writeQuery({ query: FEED_QUERY, data })
  }

  _subscribeToNewLinks = async () => {
    // ... you'll implement this 🔜
  }

  render() {
    return (
      <Query query={FEED_QUERY}>
        {({ loading, error, data, subscribeToMore }) => {
          if (loading) return <div>Fetching</div>
          if (error) return <div>Error</div>

          this._subscribeToNewLinks(subscribeToMore)
    
          const linksToRender = data.feed.links
    
          return (
            <div>
              {linksToRender.map((link, index) => (
                <Link
                  key={link.id}
                  link={link}
                  index={index}
                  updateStoreAfterVote={this._updateCacheAfterVote}
                />
              ))}
            </div>
          )
        }}
      </Query>
    )
  }
}
```

</Instruction>

让我们了解这里发生了什么!你正在使用`<Query />`组件一如既往,但现在你正在使用[`subscribeToMore`](https://www.apollographql.com/docs/react/features/subscriptions.html#subscribe-to-more)作为prop获得到组件的渲染道具功能. 调用`_subscribeToNewLinks`与它各自`subscribeToMore`功能,您确保组件实际订阅事件. 此调用将打开与订阅服务器的websocket连接. 

<Instruction>

还在`LinkList.js`实行`_subscribeToNewLinks`像这样: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
_subscribeToNewLinks = subscribeToMore => {
  subscribeToMore({
    document: NEW_LINKS_SUBSCRIPTION,
    updateQuery: (prev, { subscriptionData }) => {
      if (!subscriptionData.data) return prev
      const newLink = subscriptionData.data.newLink.node

      return Object.assign({}, prev, {
        feed: {
          links: [newLink, ...prev.feed.links],
          count: prev.feed.links.length + 1,
          __typename: prev.feed.__typename
        }
      })
    }
  })
}
```

</Instruction>

你传递了两个参数`subscribeToMore`: 

1.  `document`: 这表示订阅查询本身. 在您的情况下,每次创建新链接时都会触发订阅. 
2.  `updateQuery`: 类似于缓存`update`prop,此函数允许您确定如何使用事件发生后服务器发送的信息更新存储. 事实上,它遵循与a完全相同的原则[Redux减速机](http://redux.js.org/docs/basics/Reducers.html): 它将以前的状态 (查询的状态) 作为参数`subscribeToMore`被调用) 和服务器发送的订阅数据. 然后,您可以确定如何将订阅数据合并到现有状态并返回更新的数据. 你在里面做的一切`updateQuery`从收到的内容中检索新链接`subscriptionData`,将其合并到现有链接列表中并返回此操作的结果. 

<Instruction>

你需要做的最后一件事就是添加`NEW_LINKS_SUBSCRIPTION`到文件的顶部: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
const NEW_LINKS_SUBSCRIPTION = gql`
  subscription {
    newLink {
      node {
        id
        url
        description
        createdAt
        postedBy {
          id
          name
        }
        votes {
          id
          user {
            id
          }
        }
      }
    }
  }
`
```

</Instruction>

太棒了,就是这样!您可以通过打开两个浏览器窗口来测试您的实现. 在第一个窗口中,您运行应用程序`http://localhost:3000/`. 用于打开Playground并发送的第二个窗口`post`突变. 当您发送变异时,您会看到应用程序实时更新!⚡️

> **注意**: 目前有一个[窃听器](https://github.com/apollographql/apollo-link/issues/428)在里面`apollo-link-ws`由于以下错误而阻止您的应用运行的软件包: `Module not found: Can't resolve 'subscriptions-transport-ws' in '/.../hackernews-react-apollo/node_modules/apollo-link-ws/lib'`解决方法直到修复为手动安装`subscriptions-transport-ws`包装`yarn add subscriptions-transport-ws`. 还有另一个[窃听器](https://github.com/graphcool/graphql-yoga/issues/101)在`graphql-yoga`这会导致订阅多次触发 (而链接实际上只创建一次) . 重新加载网站后,您将看到正确数量的链接. 

### 订阅新票

接下来,您将订阅其他用户提交的新投票,以便在应用中始终显示最新的投票计数. 

<Instruction>

打开`LinkList.js`并将以下方法添加到`LinkList`零件: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
_subscribeToNewVotes = subscribeToMore => {
  subscribeToMore({
    document: NEW_VOTES_SUBSCRIPTION
  })
}
```

</Instruction>

和以前一样,你正在打电话`subscribeToMore`但现在使用`NEW_VOTES_SUBSCRIPTION`作为文件. 这次你传递的订阅要求新创建的投票. 当订阅触发时,Apollo Client会自动更新已投票的链接. 

<Instruction>

还在`LinkList.js`添加`NEW_VOTES_SUBSCRIPTION`到文件的顶部: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
const NEW_VOTES_SUBSCRIPTION = gql`
  subscription {
    newVote {
      node {
        id
        link {
          id
          url
          description
          createdAt
          postedBy {
            id
            name
          }
          votes {
            id
            user {
              id
            }
          }
        }
        user {
          id
        }
      }
    }
  }
`
```

</Instruction>

<Instruction>

最后,继续打电话`_subscribeToNewVotes`内`render`你也做过`_subscribeToNewLinks`: 

```js{2}(path=".../hackernews-react-apollo/src/components/LinkList.js")
this._subscribeToNewLinks(subscribeToMore)
this._subscribeToNewVotes(subscribeToMore)
```

</Instruction>

太棒了!您的应用现在已准备好实时,并会在其他用户创建时立即更新链接和投票. 
