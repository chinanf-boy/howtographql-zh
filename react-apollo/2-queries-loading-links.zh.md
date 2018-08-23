---
title: "Queries: Loading Links"
pageTitle: "Fetching Data using GraphQL Queries with React & Apollo Tutorial"
description: "Learn how you can use GraphQL queries with Apollo Client to load data from a server and display it in your React components."
question: What's the declarative way for loading data with React & Apollo?
answers: ["Using a higher-order component called 'graphql'", "Using the '<Query />' component and passing GraphQL query as prop", "Using 'fetch' and putting the query in the body of the request", "Using XMLHTTPRequest and putting the query in the body of the request"]
correctAnswer: 1
videoId: ""
duration: 0		
videoAuthor: ""
---
### 准备React组件

您将在应用程序中实现的第一项功能是加载并显示一个列表`Link`元素. 您将在React组件层次结构中向前走,并从将呈现单个链接的组件开始. 

<Instruction>

创建一个名为的新文件`Link.js`在里面`components`目录并添加以下代码: 

```js(path=".../hackernews-react-apollo/src/components/Link.js")
import React, { Component } from 'react'

class Link extends Component {
  render() {
    return (
      <div>
        <div>
          {this.props.link.description} ({this.props.link.url})
        </div>
      </div>
    )
  }
}

export default Link
```

</Instruction>

这是一个简单的React组件,需要一个`link`在它的`props`并呈现链接`description`和`url`. 易如反掌!🍰

接下来,您将实现呈现链接列表的组件. 

<Instruction>

再次,在`components`目录,继续创建一个名为的新文件`LinkList.js`. 然后添加以下代码: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
import React, { Component } from 'react'
import Link from './Link'

class LinkList extends Component {
  render() {
    const linksToRender = [
      {
        id: '1',
        description: 'Prisma turns your database into a GraphQL API 😎 😎',
        url: 'https://www.prismagraphql.com',
      },
      {
        id: '2',
        description: 'The best GraphQL client',
        url: 'https://www.apollographql.com/docs/react/',
      },
    ]

    return (
      <div>{linksToRender.map(link => <Link key={link.id} link={link} />)}</div>
    )
  }
}

export default LinkList
```

</Instruction>

在这里,您现在使用本地模拟数据来确保组件设置正常工作. 你很快就会用从服务器加载的一些实际数据来代替它 - 耐心,年轻的Padawan!

<Instruction>

要完成设置,请打开`App.js`并用以下内容替换当前内容: 

```js(path=".../hackernews-react-apollo/src/components/App.js")
import React, { Component } from 'react'
import LinkList from './LinkList'

class App extends Component {
  render() {
    return <LinkList />
  }
}

export default App
```

</Instruction>

运行应用程序以检查一切是否有效!该应用程序现在应显示来自的两个链接`linksToRender`数组: 

![](https://imgur.com/VJzRyjq.png)

### 编写GraphQL查询

接下来,您将加载存储在数据库中的实际链接. 您需要做的第一件事是定义要发送给API的GraphQL查询. 

这是它的样子: 

```graphql
{
  feed {
    links {
      id
      createdAt
      description
      url
    }
  }
}
```

您现在可以简单地执行此查询[操场](https://www.prisma.io/docs/graphql-ecosystem/graphql-playground/overview-chaha125ho) (反对这*应用模式*) 并从GraphQL服务器检索结果. 但是如何在JavaScript代码中使用它?

### 使用Apollo客户端查询

使用Apollo时,您有两种向服务器发送查询的方法. 

第一个是直接使用[询问](https://www.apollographql.com/docs/react/api/apollo-client.html#ApolloClient.query)关于的方法`ApolloClient`直. 这是一种非常直接的获取数据的方法,并允许您将响应处理为*诺言*. 

一个实际的例子如下: 

```js(nocopy)
client.query({
  query: gql`
    {
      feed {
        links {
          id
        }
      }
    }
  `
}).then(response => console.log(response.data.allLinks))
```

然而,使用React时更具声明性的方法是使用新的Apollo[渲染道具API](https://dev-blog.apollodata.com/introducing-react-apollo-2-1-c837cc23d926)仅使用组件来管理GraphQL数据. 

使用这种方法,在数据提取方面您需要做的就是将GraphQL查询作为prop和`<Query />`组件将为您提取数据,然后它将使其在组件中可用[渲染道具功能](https://reactjs.org/docs/render-props.html). 

通常,每次添加一些数据获取逻辑的过程都非常相似: 

1.  使用. 将查询写为JavaScript常量`gql`解析器功能
2.  使用`<Query />`组件将GraphQL查询作为prop传递
3.  访问注入组件的查询结果`render prop function`

<Instruction>

打开`LinkList.js`并将查询添加到文件的顶部: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
const FEED_QUERY = gql`
  {
    feed {
      links {
        id
        createdAt
        url
        description
      }
    }
  }
`
```

</Instruction>
<Instruction>

也替换当前`return`以下内容: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
return (
  <Query query={FEED_QUERY}>
    {() => linksToRender.map(link => <Link key={link.id} link={link} />)}
  </Query>
)
```

</Instruction>

这里发生了什么?

1.  首先,创建名为的JavaScript常量`FEED_QUERY`存储查询. 该`gql`function用于解析包含GraphQL代码的纯字符串 (如果你不熟悉反引号语法,你可以阅读JavaScript的[标记的模板文字](http://wesbos.com/tagged-template-literals/)) . 
2.  最后,使用包装返回的代码`<Query />`组件传递`FEED_QUERY`作为道具. 

> **注意**: 请注意,我们正在返回`linksToRender`作为一个功能结果,这是由于`render prop function`由...提供`<Query />`零件. 

<Instruction>

要使此代码起作用,您还需要导入相应的依赖项. 将以下两行添加到文件的顶部,在其他import语句的正下方: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
import { Query } from 'react-apollo'
import gql from 'graphql-tag'
```

</Instruction>

太棒了,这就是你所有的数据提取代码,你能相信吗?但正如你所看到的,它没有接收服务器数据,所以让它成为现实吧

您现在可以最终删除模拟数据并渲染从服务器获取的实际链接,这要归功于`<Query />`渲染道具功能. 

<Instruction>

还在`LinkList.js`,更新`component`如下: 

```js{6,7,9}(path=".../hackernews-react-apollo/src/components/LinkList.js")
class LinkList extends Component {
  render() {
    return (
      <Query query={FEED_QUERY}>
        {({ loading, error, data }) => {
          if (loading) return <div>Fetching</div>
          if (error) return <div>Error</div>
    
          const linksToRender = data.feed.links
    
          return (
            <div>
              {linksToRender.map(link => <Link key={link.id} link={link} />)}
            </div>
          )
        }}
      </Query>
    )
  }
}
```

</Instruction>

让我们来看看这段代码中发生的事情. 正如所料,Apollo在组件中注入了几个道具`render prop function`. 这些道具本身提供有关的信息*州*网络请求: 

1.  `loading`: 是的`true`只要请求仍在进行且未收到响应. 
2.  `error`: 如果请求失败,此字段将包含有关确切错误的信息. 
3.  `data`: 这是从服务器收到的实际数据. 它有`links`表示列表的属性`Link`元素. 

> 实际上,注入的道具包含更多功能. 你可以阅读更多内容[API概述](https://www.apollographql.com/docs/react/essentials/queries.html#render-prop). 

而已!您应该看到与以前完全相同的屏幕. 

> **注意**: 如果浏览器开启`http://localhost:4000`只说错误而且是空的,否则你可能忘了让你的服​​务器运行. 请注意,要使应用程序正常运行,服务器也需要运行 - 因此终端中有两个正在运行的进程: 一个用于服务器,一个用于React应用程序. 要启动服务器,请导航到`server`目录并运行`yarn start`. 
