---
title: More Mutations and Updating the Store
pageTitle: "Mutations & Caching with GraphQL, React & Apollo Tutorial"
description: "Learn how to use Apollo's imperative store API to update the cache after a GraphQL mutation. The updates will automatically end up in your React components."
question: "What does the 'update' prop of Apollo's <Mutation /> component do?"
answers: ["It allows to update your Apollo Client dependency locally", "It allows to update the local Apollo store in a declarative fashion", "It allows to update your store based on your mutation’s result", "It updates the GraphQL schema locally so Apollo Client can verify your queries and mutations before they're sent to the server"]
correctAnswer: 2
videoId: ""
duration: 0		
videoAuthor: ""
---
您要实现的下一个功能是投票功能!经过身份验证的用户可以提交链接投票. 最受欢迎的链接稍后将显示在单独的路线上!

### 准备React组件

再一次,实现此新功能的第一步是使您的React组件为预期的功能做好准备. 

</Instruction>

打开`Link.js`并更新`render`看起来如下: 

```js(path=".../hackernews-react-apollo/src/components/Link.js")
render() {
  const authToken = localStorage.getItem(AUTH_TOKEN)
  return (
    <div className="flex mt2 items-start">
      <div className="flex items-center">
        <span className="gray">{this.props.index + 1}.</span>
        {authToken && (
          <div className="ml1 gray f11" onClick={() => this._voteForLink()}>
            ▲
          </div>
        )}
      </div>
      <div className="ml1">
        <div>
          {this.props.link.description} ({this.props.link.url})
        </div>
        <div className="f6 lh-copy gray">
          {this.props.link.votes.length} votes | by{' '}
          {this.props.link.postedBy
            ? this.props.link.postedBy.name
            : 'Unknown'}{' '}
          {timeDifferenceForDate(this.props.link.createdAt)}
        </div>
      </div>
    </div>
  )
}
```

</Instruction>

你已经准备好了`Link`组件,用于呈现每个链接的投票数以及发布它的用户的名称. 此外,如果用户当前已登录,您将呈现upvote按钮 - 这就是您正在使用的按钮`authToken`对于. 如果`Link`与a无关`User`,用户名将显示为`Unknown`. 

请注意,您还使用了一个名为的函数`timeDifferenceForDate`通过了`createdAt`每个链接的信息. 该函数将采用时间戳并将其转换为更加用户友好的字符串,例如`"3 hours ago"`. 

继续实施`timeDifferenceForDate`函数接下来,你可以导入并使用它`Link`零件. 

<Instruction>

创建一个名为的新文件`utils.js`在里面`src`目录并将以下代码粘贴到其中: 

```js(path=".../hackernews-react-apollo/src/utils.js")
function timeDifference(current, previous) {
  const milliSecondsPerMinute = 60 * 1000
  const milliSecondsPerHour = milliSecondsPerMinute * 60
  const milliSecondsPerDay = milliSecondsPerHour * 24
  const milliSecondsPerMonth = milliSecondsPerDay * 30
  const milliSecondsPerYear = milliSecondsPerDay * 365

  const elapsed = current - previous

  if (elapsed < milliSecondsPerMinute / 3) {
    return 'just now'
  }

  if (elapsed < milliSecondsPerMinute) {
    return 'less than 1 min ago'
  } else if (elapsed < milliSecondsPerHour) {
    return Math.round(elapsed / milliSecondsPerMinute) + ' min ago'
  } else if (elapsed < milliSecondsPerDay) {
    return Math.round(elapsed / milliSecondsPerHour) + ' h ago'
  } else if (elapsed < milliSecondsPerMonth) {
    return Math.round(elapsed / milliSecondsPerDay) + ' days ago'
  } else if (elapsed < milliSecondsPerYear) {
    return Math.round(elapsed / milliSecondsPerMonth) + ' mo ago'
  } else {
    return Math.round(elapsed / milliSecondsPerYear) + ' years ago'
  }
}

export function timeDifferenceForDate(date) {
  const now = new Date().getTime()
  const updated = new Date(date).getTime()
  return timeDifference(now, updated)
}
```

</Instruction>

<Instruction>

早在`Link.js`,进口`AUTH_TOKEN`和`timeDifferenceForDate`在文件顶部: 

```js(path=".../hackernews-react-apollo/src/components/Link.js")
import { AUTH_TOKEN } from '../constants'
import { timeDifferenceForDate } from '../utils'
```

</Instruction>

最后,每个`Link`element也会在列表中呈现它的位置,所以你必须传递一个`index`来自`LinkList`零件. 

<Instruction>

打开`LinkList.js`并更新的渲染`Link`里面的组件`render`还包括链接的位置: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
return (
  <div>
    {linksToRender.map((link, index) => (
      <Link key={link.id} link={link} index={index} />
    ))}
  </div>
)
```

</Instruction>

请注意,该应用程序将不会在此时运行`votes`尚未包含在查询中. 你接下来就解决了!

<Instruction>

打开`LinkList.js`并更新的定义`FEED_QUERY`看起来如下: 

```js{9-18}(path=".../hackernews-react-apollo/src/components/LinkList.js")
const FEED_QUERY = gql`
  {
    feed {
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
    }
  }
`
```

</Instruction>

您在此处所做的只是包含有关发布链接的用户的信息以及有关链接在查询的有效负载中的投票的信息. 您现在可以再次运行该应用程序,并正确显示链接. 

![](https://imgur.com/tKzj3b5.png)

让我们继续前进并实施`vote`突变!

### 呼唤变异

<Instruction>

打开`Link.js`并将以下变异定义添加到文件的顶部. 

```js(path=".../hackernews-react-apollo/src/components/Link.js")
const VOTE_MUTATION = gql`
  mutation VoteMutation($linkId: ID!) {
    vote(linkId: $linkId) {
      id
      link {
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
`
```

</Instruction>

<Instruction>

再次,也取代了当前`flex items-center`班级名称`div`元素如下: 

```js{4-10}(path=".../hackernews-react-apollo/src/components/Link.js")
<div className="flex items-center">
  <span className="gray">{this.props.index + 1}.</span>
  {authToken && (
    <Mutation mutation={VOTE_MUTATION} variables={{ linkId: this.props.link.id }}>
      {voteMutation => (
        <div className="ml1 gray f11" onClick={voteMutation}>
          ▲
        </div>
      )}
    </Mutation>
  )}
</div>
```

</Instruction>

现在这一步应该让人感觉很熟悉. 你正在添加调用的能力`voteMutation`通过使用我们的功能组件`<Mutation />`组件 (我们也过去了`VOTE_MUTATION`和`link.id`作为道具) . 

<Instruction>

同样,您需要导入`Mutation`和`graphql`在...之上`Link.js`文件: 

```js(path=".../hackernews-react-apollo/src/components/Link.js")
import { Mutation } from 'react-apollo'
import gql from 'graphql-tag'
```

</Instruction>

您现在可以去测试实现了!跑`yarn start`在`hackernews-react-apollo`并单击链接上的upvote按钮. 你还没有得到任何UI反馈,但刷新页面后你会看到增加的投票. 

在下一节中,您将学习如何在每次突变后自动更新UI!

### 更新缓存

关于Apollo的一个很酷的事情是你可以手动控制缓存的内容. 这非常方便,尤其是在进行突变后. 它允许精确确定您希望如何更新缓存. 在这里,您将使用它来确保UI在显示之后立即显示正确的投票数`vote`进行了突变. 

您将使用Apollo实现此功能[缓存数据](https://www.apollographql.com/docs/react/advanced/caching.html#after-mutations). 

<Instruction>

打开`Link.js`并替换`<Mutation />`组件添加`update`这样的道具: 

```js{4-6}(path=".../hackernews-react-apollo/src/components/Link.js")
<Mutation
  mutation={VOTE_MUTATION}
  variables={{ linkId: this.props.link.id }}
  update={(store, { data: { vote } }) =>
    this.props.updateStoreAfterVote(store, vote, this.props.link.id)
  }
>
  {voteMutation => (
    <div className="ml1 gray f11" onClick={voteMutation}>
      ▲
    </div>
  )}
</Mutation>
```

</Instruction>

该`update`你作为道具传递的功能`<Mutation />`在服务器返回响应后,将直接调用组件. 它接收突变的有效载荷 (`data`) 和当前缓存 (`store`作为论据. 然后,您可以使用此输入来确定缓存的新状态. 

请注意,你已经是[解构](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)服务器响应和检索`vote`从它的领域. 

好的,所以现在你知道这是什么了`update`函数是,但实际的实现将在父组件中完成`Link`,是的`LinkList`. 

<Instruction>

打开`LinkList.js`并在范围内添加以下方法`LinkList`零件: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
_updateCacheAfterVote = (store, createVote, linkId) => {
  const data = store.readQuery({ query: FEED_QUERY })

  const votedLink = data.feed.links.find(link => link.id === linkId)
  votedLink.votes = createVote.link.votes

  store.writeQuery({ query: FEED_QUERY, data })
}
```

</Instruction>

这里发生了什么?

1.  首先阅读缓存数据的当前状态`FEED_QUERY`来自`store`. 
2.  现在,您正在从该列表中检索用户刚刚投票的链接. 您还通过重置它来操纵该链接`votes`到了`votes`刚刚由服务器返回的. 
3.  最后,您获取修改后的数据并将其写回商店. 

接下来,您需要将此功能传递给`Link`所以它可以从那里调用. 

<Instruction>

还在`LinkList.js`,更新方式如何`Link`返回组件: 

```js{5-5}(path=".../hackernews-react-apollo/src/components/LinkList.js")
<Link
  key={link.id}
  link={link}
  index={index}
  updateStoreAfterVote={this._updateCacheAfterVote}
/>
```

</Instruction>

而已!该`update`现在将执行函数并确保在执行突变后存储正确更新. 商店更新将触发组件的重新呈现,从而使用正确的信息更新UI!

虽然我们正在努力,但我们也要实施`update`添加新链接!

<Instruction>

打开`CreateLink.js`我们之前做过,更新`<Mutation />`组件传递`update`像这样的道具: 

```js{5-12}(path=".../hackernews-react-apollo/src/components/CreateLink.js")
<Mutation
  mutation={POST_MUTATION}
  variables={{ description, url }}
  onCompleted={() => this.props.history.push('/')}
  update={(store, { data: { post } }) => {
    const data = store.readQuery({ query: FEED_QUERY })
    data.feed.links.unshift(post)
    store.writeQuery({
      query: FEED_QUERY,
      data
    })
  }}
>
  {postMutation => <button onClick={postMutation}>Submit</button>}
</Mutation>
```

</Instruction>

该`update`功能以与以前非常相似的方式工作. 您首先要读取结果的当前状态`FEED_QUERY`. 然后在开头插入最新的链接,并将查询结果写回商店. 

<Instruction>

你需要做的最后一件事是导入`FEED_QUERY`进入该文件: 

```js(path=".../hackernews-react-apollo/src/components/CreateLink.js")
import { FEED_QUERY } from './LinkList'
```

</Instruction>

相反,它也需要从定义的位置导出. 

<Instruction>

打开`LinkList.js`并调整的定义`FEED_QUERY`通过添加`export`关键字: 

```js(path=".../hackernews-react-apollo/src/components/LinkList.js")
export const FEED_QUERY = ...
```

</Instruction>

很棒,现在商店也会在添加新链接后使用正确的信息进行更新. 该应用程序正在形成🤓
