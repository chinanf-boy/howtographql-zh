---
title: Pagination
pageTitle: "Pagination with GraphQL, React & Apollo Tutorial"
description: "Learn how to implement limit-offset pagination with GraphQL and Apollo Client in a React app. The Prisma API exposes the required arguments for lists."
question: "What's the difference between the 'query' and 'readQuery' methods on the 'ApolloClient'?"
answers: ["'readQuery' always fetches data over the network while 'query' can retrieve data either from the cache or remotely", "'readQuery' can only be used to reading data while 'query' can also be used to write data", "'readQuery' was formerly called 'query' and the functionality of both is identical", "'readQuery' always reads data from the local cache while 'query' might retrieve data either from the cache or remotely"]
correctAnswer: 3
videoId: ""
duration: 0		
videoAuthor: ""
---
我们将在本教程中介绍的最后一个主题是分页. 您将实现一种简单的分页方法,以便用户能够以较小的块查看链接,而不是拥有一个非常长的列表`Link`元素. 

## 准备React组件

再一次,您首先需要为此新功能准备React组件. 实际上,我们会稍微调整当前的路由设置. 这是一个想法: The`LinkList`组件将用于两个不同的用例 (和路由) . 第一个是显示10个最高投票链接. 它的第二个用例是在列表中显示新链接,这些链接分为多个页面,用户可以浏览这些页面. 

<Instruction>

打开`App.js`并像这样调整渲染方法: 

```js{4,8,9}(path=".../hackernews-react-apollo/src/components/App.js")
render() {
  return (
    <div className='center w85'>
      <Header />
      <div className='ph3 pv1 background-gray'>
        <Switch>
          <Route exact path='/' render={() => <Redirect to='/new/1' />} />
          <Route exact path='/create' component={CreateLink} />
          <Route exact path='/login' component={Login} />
          <Route exact path='/search' component={Search} />
          <Route exact path='/top' component={LinkList} />
          <Route exact path='/new/:page' component={LinkList} />
        </Switch>
      </div>
    </div>
  )
}
```

</Instruction>

确保导入`Redirect`组件,所以你不会得到任何错误. 

<Instruction>

更新文件顶部的路由器导入: 

```js(path=".../hackernews-react-apollo/src/components/App.js")
import { Switch, Route, Redirect } from 'react-router-dom'
```

</Instruction>

您现在添加了两条新路线: `/top`和`/new/:page`. 后者读取的值`page`来自url,以便在呈现的组件内部提供此信息,此处为`LinkList`. 

根路线`/`现在重定向到显示新帖子的路线的第一页. 

在继续之前,快速添加一个新的导航项目`Header`将用户带到的组件`/top`路线. 

<Instruction>

打开`Header.js`添加以下行*之间*该`/`和`/search`路线: 

```js(path=".../hackernews-react-apollo/src/components/Header.js")
<Link to="/top" className="ml1 no-underline black">
  top
</Link>
<div className="ml1">|</div>
```

</Instruction>

你还需要添加一些逻辑`LinkList`组件,以说明它现在有两个不同的职责. 

<Instruction>

打开`LinkList.js`并添加三个参数`FeedQuery`通过替换`FEED_QUERY`定义如下: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
export const FEED_QUERY = gql`
  query FeedQuery($first: Int, $skip: Int, $orderBy: LinkOrderByInput) {
    feed(first: $first, skip: $skip, orderBy: $orderBy) {
      links {
        id
        createdAt
        url
        description
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
      count
    }
  }
`
```

</Instruction>

查询现在接受我们将用于实现分页和排序的参数. `skip`定义了*抵消*查询将从何处开始. 如果您传递了例如`10`对于此参数,这意味着列表的前10项不会包含在响应中. `first`然后定义*限制*, 要么*多少*要从该列表加载的元素. 说,你通过了`10`对于`skip`和`5`对于`first`,您将从列表中收到第10至15项. `orderBy`定义如何对返回的列表进行排序. 

但是我们如何在使用时传递变量`<Query />`在引擎盖下获取数据的组件?你需要提供参数`variables`在你声明组件的地方. 

<Instruction>

还在`LinkList.js`,在LinkList组件的范围内添加以下方法: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
_getQueryVariables = () => {
  const isNewPage = this.props.location.pathname.includes('new')
  const page = parseInt(this.props.match.params.page, 10)

  const skip = isNewPage ? (page - 1) * LINKS_PER_PAGE : 0
  const first = isNewPage ? LINKS_PER_PAGE : 100
  const orderBy = isNewPage ? 'createdAt_DESC' : null
  return { first, skip, orderBy }
}
```

</Instruction>

<Instruction>

并更新`<Query />`组件定义如下: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
<Query query={FEED_QUERY} variables={this._getQueryVariables()}>
```

</Instruction>

你现在正在路过`first, skip, orderBy`价值为`variables`基于当前页面 (`this.props.match.params.page`) 它用于计算您检索的链接块. 

另请注意,您要包含订购属性`createdAt_DESC`为了`new`页面,以确保首先显示最新的链接. 订购的`/top`路线将根据每个链接的投票数手动计算. 

你还需要定义`LINKS_PER_PAGE`常数然后将其导入到`LinkList`零件. 

<Instruction>

打开`src/constants.js`并添加以下定义: 

```js(path=".../hackernews-react-apollo/src/constants.js")
export const LINKS_PER_PAGE = 5
```

</Instruction>

### 实施导航

接下来,您需要用户在页面之间切换的功能. 首先加两个`button`元素到底部`LinkList`可用于来回导航的组件. 

<Instruction>

打开`LinkList.js`并更新`render`看起来如下: 

```js{11-15,27-36}(path=".../hackernews-react-apollo/src/components/LinkList.js")
render() {
  return (
    <Query query={FEED_QUERY} variables={this._getQueryVariables()}>
      {({ loading, error, data, subscribeToMore }) => {
        if (loading) return <div>Fetching</div>
        if (error) return <div>Error</div>

        this._subscribeToNewLinks(subscribeToMore)
        this._subscribeToNewVotes(subscribeToMore)

        const linksToRender = this._getLinksToRender(data)
        const isNewPage = this.props.location.pathname.includes('new')
        const pageIndex = this.props.match.params.page
          ? (this.props.match.params.page - 1) * LINKS_PER_PAGE
          : 0

        return (
          <Fragment>
            {linksToRender.map((link, index) => (
              <Link
                key={link.id}
                link={link}
                index={index + pageIndex}
                updateStoreAfterVote={this._updateCacheAfterVote}
              />
            ))}
            {isNewPage && (
              <div className="flex ml4 mv3 gray">
                <div className="pointer mr2" onClick={this._previousPage}>
                  Previous
                </div>
                <div className="pointer" onClick={() => this._nextPage(data)}>
                  Next
                </div>
              </div>
            )}
          </Fragment>
        )
      }}
    </Query>
  )
}
```

</Instruction>

<Instruction>

现在调整import语句`../constants`在`LinkList.js`还包括新常量: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
import { LINKS_PER_PAGE } from '../constants'
```

</Instruction>

在继续之前,发现[反应碎片](https://reactjs.org/docs/fragments.html)组件返回多个元素而不向DOM添加额外节点的常见模式

<Instruction>

别忘了添加`Fragment`到文件的顶部: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
import React, { Component, Fragment } from 'react'
```

</Instruction>

由于现在设置稍微复杂一些,因此您将计算要在单独方法中呈现的链接列表. 

<Instruction>

还在`LinkList.js`,添加以下方法实现: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
_getLinksToRender = data => {
  const isNewPage = this.props.location.pathname.includes('new')
  if (isNewPage) {
    return data.feed.links
  }
  const rankedLinks = data.feed.links.slice()
  rankedLinks.sort((l1, l2) => l2.votes.length - l1.votes.length)
  return rankedLinks
}
```

</Instruction>

为了`newPage`,您只需返回查询返回的所有链接. 这是合乎逻辑的,因为在这里您不必对要呈现的列表进行任何手动修改. 如果用户从中加载了组件`/top`路线,您将根据投票数对列表进行排序并返回前10个链接. 

接下来,您将实现的功能*以前*和*下一个*纽扣. 

<Instruction>

在`LinkList.js`,添加以下两个按下按钮时将调用的方法: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
_nextPage = data => {
  const page = parseInt(this.props.match.params.page, 10)
  if (page <= data.feed.count / LINKS_PER_PAGE) {
    const nextPage = page + 1
    this.props.history.push(`/new/${nextPage}`)
  }
}

_previousPage = () => {
  const page = parseInt(this.props.match.params.page, 10)
  if (page > 1) {
    const previousPage = page - 1
    this.props.history.push(`/new/${previousPage}`)
  }
}
```

</Instruction>

这些的实现非常简单. 您正在从网址检索当前页面并执行完整性检查,以确保来回分页是有意义的. 然后,您只需计算下一页并告诉路由器接下来要导航的位置. 然后路由器将使用新的重新加载组件`page`在url中,将用于计算要加载的正确链接块. 键入运行应用程序`yarn start`在终端中使用新按钮分页链接!

### 最后的调整

通过我们所做的改变`FEED_QUERY`,你会注意到的`update`你的突变功能不再起作用了. 那是因为`readQuery`现在也期望传递我们之前定义的相同变量. 

> **注意**: `readQuery`基本上与工作方式相同`query`关于的方法`ApolloClient`你曾经用来实现搜索. 但是,它不是调用服务器,而是简单地解析对本地存储的查询!如果使用变量从服务器获取查询,`readQuery`还需要知道变量以确保它可以从缓存中提供正确的信息. 

<Instruction>

有了这些信息,请打开`LinkList.js`并更新`_updateCacheAfterVote`方法如下: 

```js{2-11}(path=".../hackernews-react-apollo/src/components/LinkList.js")
_updateCacheAfterVote = (store, createVote, linkId) => {
  const isNewPage = this.props.location.pathname.includes('new')
  const page = parseInt(this.props.match.params.page, 10)

  const skip = isNewPage ? (page - 1) * LINKS_PER_PAGE : 0
  const first = isNewPage ? LINKS_PER_PAGE : 100
  const orderBy = isNewPage ? 'createdAt_DESC' : null
  const data = store.readQuery({
    query: FEED_QUERY,
    variables: { first, skip, orderBy }
  })

  const votedLink = data.feed.links.find(link => link.id === linkId)
  votedLink.votes = createVote.link.votes
  store.writeQuery({ query: FEED_QUERY, data })
}
```

</Instruction>

这里发生的一切都是变量的计算取决于用户当前是否在`/top`要么`/new`路线. 

最后,您还需要调整执行`update`何时创建新链接. 

<Instruction>

打开`CreateLink.js`并替换当前`<Mutation />`组件如此: 

```js{4-19}(path=".../hackernews-react-apollo/src/components/CreateLink.js")
<Mutation
  mutation={POST_MUTATION}
  variables={{ description, url }}
  onCompleted={() => this.props.history.push('/new/1')}
  update={(store, { data: { post } }) => {
    const first = LINKS_PER_PAGE
    const skip = 0
    const orderBy = 'createdAt_DESC'
    const data = store.readQuery({
      query: FEED_QUERY,
      variables: { first, skip, orderBy }
    })
    data.feed.links.unshift(post)
    store.writeQuery({
      query: FEED_QUERY,
      data,
      variables: { first, skip, orderBy }
    })
  }}
>
  {postMutation => <button onClick={postMutation}>Submit</button>}
</Mutation>
```

</Instruction>

<Instruction>

既然你没有`LINKS_PER_PAGE`此组件中的常量可用,请确保将其导入到文件的顶部: 

```js(path=".../hackernews-react-apollo/src/components/CreateLink.js")
import { LINKS_PER_PAGE } from '../constants'
```

</Instruction>

您现在已经为应用添加了一个简单的分页系统,允许用户以小块的形式加载链接,而不是预先加载它们. 
