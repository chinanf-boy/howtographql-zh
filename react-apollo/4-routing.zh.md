---
title: Routing
pageTitle: "React Router with GraphQL & Apollo Tutorial"
description: "Learn how to use react-router 4 together with GraphQL and Apollo Client to implement navigation in a React app. Each route will be represented as a `Link`."
question: What's the role of the Link component that you added in this chapter?
answers: ["It renders a link that was posted by a user", "It renders the input form for users to create new links", "It lets you navigate to a different URL", "It links your root component with all its children"]
correctAnswer: 2
videoId: ""
duration: 0		
videoAuthor: ""
---
在本节中,您将学习如何使用[反应路由器](https://github.com/ReactTraining/react-router)使用Apollo库实现一些导航功能!

### 安装依赖项

首先将所需的依赖项添加到应用程序. 打开终端,导航到项目目录并键入: 

<Instruction>

```bash(path=".../hackernews-react-apollo")
yarn add react-router react-router-dom
```

</Instruction>

### 创建标题

在继续为您的应用程序配置不同的路由之前,您需要创建一个`Header`用户可以用来在应用的不同部分之间导航的组件. 

<Instruction>

在中创建一个新文件`src/components`并称之为`Header.js`. 然后将以下代码粘贴到其中: 

```js(path=".../hackernews-react-apollo/src/components/Header.js")
import React, { Component } from 'react'
import { Link } from 'react-router-dom'
import { withRouter } from 'react-router'

class Header extends Component {
  render() {
    return (
      <div className="flex pa1 justify-between nowrap orange">
        <div className="flex flex-fixed black">
          <div className="fw7 mr1">Hacker News</div>
          <Link to="/" className="ml1 no-underline black">
            new
          </Link>
          <div className="ml1">|</div>
          <Link to="/create" className="ml1 no-underline black">
            submit
          </Link>
        </div>
      </div>
    )
  }
}

export default withRouter(Header)
```

</Instruction>

这简单地呈现两个`Link`用户可以用来在组件之间导航的组件`LinkList`和`CreateLink`组件. 

> 不要被"其他"搞糊涂`Link`这里使用的组件. 你正在使用的那个`Header`与此无关`Link`你之前写过的组件,它们碰巧具有相同的名称. 这个[链接](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/docs/api/Link.md)源于`react-router-dom`包,允许您在应用程序内部的路径之间导航. 

### 设置路线

您将在项目的根组件中为应用程序配置不同的路由: `App`. 

<Instruction>

打开相应的文件`App.js`并更新`render`包括`Header`以及`LinkList`和`CreateLink`不同路线下的组件: 

```js(path=".../hackernews-react-apollo/src/components/App.js")
render() {
  return (
    <div className="center w85">
      <Header />
      <div className="ph3 pv1 background-gray">
        <Switch>
          <Route exact path="/" component={LinkList} />
          <Route exact path="/create" component={CreateLink} />
        </Switch>
      </div>
    </div>
  )
}
```

</Instruction>

要使此代码起作用,您需要导入所需的依赖项`react-router-dom`. 

<Instruction>

将以下语句添加到文件的顶部: 

```js(path=".../hackernews-react-apollo/src/components/App.js")
import Header from './Header'
import { Switch, Route } from 'react-router-dom'
```

</Instruction>

现在你需要包装`App`同`BrowserRouter`这样所有的儿童成分`App`将获得访问路由功能. 

<Instruction>

打开`index.js`并将以下import语句添加到顶部: 

```js(path=".../hackernews-react-apollo/src/index.js")
import { BrowserRouter } from 'react-router-dom'
```

</Instruction>

<Instruction>

现在更新`ReactDOM.render`并用. 包装整个应用程序`BrowserRouter`: 

```js{2,6}(path=".../hackernews-react-apollo/src/index.js")
ReactDOM.render(
  <BrowserRouter>
    <ApolloProvider client={client}>
      <App />
    </ApolloProvider>
  </BrowserRouter>,
  document.getElementById('root')
)
```

</Instruction>

而已. 如果再次运行该应用,您现在可以访问两个网址. `http://localhost:3000/`将呈现`LinkList`和`http://localhost:3000/create`渲染`CreateLink`您刚才在上一节中写过的组件. 

![](https://imgur.com/X9bmkQH.png)

### 实施导航

要结束本节,您需要实现自动重定向`CreateLink`至`LinkList`在进行突变后. 

<Instruction>

打开`CreateLink.js`并更新`<Mutation />`组件看起来如下: 

```js{4}(path=".../hackernews-react-apollo/src/components/CreateLink.js")
<Mutation
  mutation={POST_MUTATION}
  variables={{ description, url }}
  onCompleted={() => this.props.history.push('/')}
>
  {postMutation => <button onClick={postMutation}>Submit</button>}
</Mutation>
```

</Instruction>

突变后,`react-router-dom`现在将导航回到`LinkList`可在根路由上访问的组件: `/`. 
