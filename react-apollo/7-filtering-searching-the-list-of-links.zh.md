---
title: "Filtering: Searching the List of Links"
pageTitle: "Filtering with GraphQL, React & Apollo Tutorial"
description: "Learn how to use filters with GraphQL and Apollo Client. Prisma provides a powerful filter and ordering API that you'll explore in this example."
question: "What's the purpose of the 'withApollo' function?"
answers: ["You use it to send queries and mutations to a GraphQL server", "When wrapped around a component, it injects the 'ApolloClient' instance into the component's props", "You have to use it everywhere where you want to use Apollo functionality", "It parses GraphQL code"]
correctAnswer: 1
---
在本节中,您将实现搜索功能并了解GraphQL API的过滤功能. 

### 准备React组件

搜索将在新路由下提供,并在新的React组件中实现. 

<Instruction>

首先创建一个名为的新文件`Search.js`在`src/components`并添加以下代码: 

```js(path=".../hackernews-react-apollo/src/components/Search.js")
import React, { Component } from 'react'
import { withApollo } from 'react-apollo'
import gql from 'graphql-tag'
import Link from './Link'

class Search extends Component {

  state = {
    links: [],
    filter: ''
  }

  render() {
    return (
      <div>
        <div>
          Search
          <input
            type='text'
            onChange={e => this.setState({ filter: e.target.value })}
          />
          <button onClick={() => this._executeSearch()}>OK</button>
        </div>
        {this.state.links.map((link, index) => (
          <Link key={link.id} link={link} index={index} />
        ))}
      </div>
    )
  }

  _executeSearch = async () => {
    // ... you'll implement this 🔜
  }
}

export default withApollo(Search)
```

</Instruction>

同样,这是一个非常标准的设置. 你正在渲染一个`input`用户可以键入搜索字符串的字段. 

请注意`links`组件状态中的字段将保存要呈现的所有链接,因此这次我们不通过组件道具访问查询结果!我们也会谈谈`withApollo`稍微导出组件时使用的函数!

<Instruction>

现在添加`Search`组件作为应用程序的新路径. 打开`App.js`并更新渲染,如下所示: 

```js{10}(path=".../hackernews-react-apollo/src/components/App.js")
render() {
  return (
    <div className='center w85'>
      <Header />
      <div className='ph3 pv1 background-gray'>
        <Switch>
          <Route exact path='/' component={LinkList} />
          <Route exact path='/create' component={CreateLink} />
          <Route exact path='/login' component={Login} />
          <Route exact path='/search' component={Search} />
        </Switch>
      </div>
    </div>
  )
}
```

</Instruction>

<Instruction>

也导入了`Search`文件顶部的组件: 

```js(path=".../hackernews-react-apollo/src/components/App.js")
import Search from './Search'
```

</Instruction>

为了使用户能够轻松导航到搜索功能,您还应该向该用户添加新的导航项`Header`. 

<Instruction>

打开`Header.js`并把一个新的`Link`之间`new`和`submit`: 

```js{6-9}(path=".../hackernews-react-apollo/src/components/Header.js")
<div className="flex flex-fixed black">
  <div className="fw7 mr1">Hacker News</div>
  <Link to="/" className="ml1 no-underline black">
    new
  </Link>
  <div className="ml1">|</div>
  <Link to="/search" className="ml1 no-underline black">
    search
  </Link>
  {authToken && (
    <div className="flex">
      <div className="ml1">|</div>
      <Link to="/create" className="ml1 no-underline black">
        submit
      </Link>
    </div>
  )}
</div>
```

</Instruction>

您现在可以使用中的新按钮导航到搜索功能`Header`: 

![](http://imgur.com/XxPdUvo.png)

好的,我们现在回到原点`Search`组件,看看我们如何实现实际搜索. 

### 过滤链接

<Instruction>

打开`Search.js`并在文件顶部添加以下查询定义: 

```js(path=".../hackernews-react-apollo/src/components/Search.js")
const FEED_SEARCH_QUERY = gql`
  query FeedSearchQuery($filter: String!) {
    feed(filter: $filter) {
      links {
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

此查询看起来类似于`feed`查询中使用的`LinkList`. 但是,这一次它引入了一个名为的论据`filter`这将用于约束您要检索的链接列表. 

实际的过滤器是在内置和使用的`feed`解析器实现于`server/src/resolvers/Query.js`: 

```js(path=".../hackernews-react-apollo/server/src/resolvers/Query.js"&nocopy)
async function feed(parent, args, ctx, info) {
  const { filter, first, skip } = args // destructure input arguments
  const where = filter
    ? { OR: [{ url_contains: filter }, { description_contains: filter }] }
    : {}

  const queriedLinks = await ctx.db.query.links({ first, skip, where })

  return {
    linkIds: queriedLinks.map(link => link.id),
    count
  }
}
```

> **注意**: 要了解此解析器中发生了什么,请查看[节点教程](https://www.howtographql.com/graphql-js/0-introduction). 

在这种情况下,两个`where`条件已指定: 只有链接才会返回`url`包含提供的`filter` *要么*它的`description`包含提供的`filter`. 这两个条件都使用Prisma的组合`OR`运营商. 

完美,查询定义!但是这次我们实际上想要在每次用户点击时加载数据**搜索**- 按钮 - 不是组件的初始加载. 

这就是目的[`withApollo`](http://dev.apollodata.com/react/higher-order-components.html#withApollo)功能. 这个函数注入了`ApolloClient`您在其中创建的实例`index.js`进入`Search`组件作为新的道具称为`client`. 

这个`client`有一个叫做的方法`query`您可以使用它来手动发送查询而不是使用`graphql`高阶分量. 

<Instruction>

这是代码的样子. 打开`Search.js`并实施`_executeSearch`如下: 

```js(path=".../hackernews-react-apollo/src/components/Search.js")
_executeSearch = async () => {
  const { filter } = this.state
  const result = await this.props.client.query({
    query: FEED_SEARCH_QUERY,
    variables: { filter },
  })
  const links = result.data.feed.links
  this.setState({ links })
}
```

</Instruction>

实施几乎是微不足道的. 你正在执行`FEED_SEARCH_QUERY`手动和检索`links`来自服务器返回的响应. 然后将这些链接放入组件中`state`这样它们就可以呈现出来. 

继续,通过运行测试应用程序`yarn start`在终端并导航到`http://localhost:3000/search`. 然后在文本字段中键入搜索字符串,单击**搜索**- 按钮并验证返回的链接是否符合过滤条件. 
