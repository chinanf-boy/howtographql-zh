---
title: Authentication
pageTitle: "Authentication with GraphQL, React & Apollo Tutorial"
description: "Learn best practices to implement authentication with GraphQL & Apollo Client to provide an email-and-password-based login in a React app with Prisma."
question: "How are HTTP requests sent by ApolloClient authenticated?"
answers: ["The ApolloClient needs to be instantiated with an authentication token", "ApolloClient exposes an extra method called 'authenticate' where you can pass an authentication token", "By attaching an authentication token to the request with dedicated ApolloLink middleware", "ApolloClient has nothing to do with authentication"]
correctAnswer: 2
videoId: ""
duration: 0		
videoAuthor: ""
---
在本节中,您将学习如何使用Apollo实现身份验证功能,以便为用户提供注册和登录功能. 

### 准备React组件

与之前的部分一样,您将通过准备此功能所需的React组件来设置登录功能的阶段. 你将从构建它开始`Login`零件. 

<Instruction>

在中创建一个新文件`src/components`并称之为`Login.js`. 然后将以下代码粘贴到其中: 

```js(path=".../hackernews-react-apollo/src/components/Login.js")
import React, { Component } from 'react'
import { AUTH_TOKEN } from '../constants'

class Login extends Component {
  state = {
    login: true, // switch between Login and SignUp
    email: '',
    password: '',
    name: '',
  }

  render() {
    const { login, email, password, name } = this.state
    return (
      <div>
        <h4 className="mv3">{login ? 'Login' : 'Sign Up'}</h4>
        <div className="flex flex-column">
          {!login && (
            <input
              value={name}
              onChange={e => this.setState({ name: e.target.value })}
              type="text"
              placeholder="Your name"
            />
          )}
          <input
            value={email}
            onChange={e => this.setState({ email: e.target.value })}
            type="text"
            placeholder="Your email address"
          />
          <input
            value={password}
            onChange={e => this.setState({ password: e.target.value })}
            type="password"
            placeholder="Choose a safe password"
          />
        </div>
        <div className="flex mt3">
          <div className="pointer mr2 button" onClick={() => this._confirm()}>
            {login ? 'login' : 'create account'}
          </div>
          <div
            className="pointer button"
            onClick={() => this.setState({ login: !login })}
          >
            {login
              ? 'need to create an account?'
              : 'already have an account?'}
          </div>
        </div>
      </div>
    )
  }

  _confirm = async () => {
    // ... you'll implement this 🔜
  }

  _saveUserData = token => {
    localStorage.setItem(AUTH_TOKEN, token)
  }
}

export default Login
```

</Instruction>

让我们快速了解这个新组件的结构,它可以有两个主要状态: 

-   一个州是**对于已拥有帐户的用户**并且只需要登录. 在此状态下,组件将仅呈现两个`input`字段供用户提供`email`和`password`. 请注意`state.login`将会`true`在这种情况下. 
-   第二个状态是**尚未创建帐户的用户**,因此仍需要注册. 在这里,你还渲染了第三个`input`用户可以提供他们的字段`name`. 在这种情况下,`state.login`将会`false`. 

方法`_confirm`将用于实现我们需要为登录功能发送的突变. 

接下来你还需要提供`constants.js`我们用来定义我们存储在浏览器中的凭据密钥的文件`localStorage`. 

> **警告**: 存储JWT`localStorage`在前端实现身份验证不是一种安全的方法. 因为本教程主要关注GraphQL,所以我们希望保持简单,因此在这里使用它. 您可以阅读有关此主题的更多信息[这里](https://www.rdegges.com/2018/please-stop-using-local-storage/). 

<Instruction>

在`src`,创建一个名为的新文件`constants.js`并添加以下定义: 

```js(path=".../hackernews-react-apollo/src/constants.js")
export const AUTH_TOKEN = 'auth-token'
```

</Instruction>

有了这个组件,你可以去添加一个新的路由`react-router-dom`建立. 

<Instruction>

打开`App.js`并更新`render`包括新路线: 

```js{7}(path=".../hackernews-react-apollo/src/components/App.js")
render() {
  return (
    <div className="center w85">
      <Header />
      <div className="ph3 pv1 background-gray">
        <Switch>
          <Route exact path="/" component={LinkList} />
          <Route exact path="/create" component={CreateLink} />
          <Route exact path="/login" component={Login} />
        </Switch>
      </div>
    </div>
  )
}
```

</Instruction>

<Instruction>

也导入了`Login`组件位于同一文件的顶部: 

```js(path=".../hackernews-react-apollo/src/components/App.js")
import Login from './Login'
```

</Instruction>

最后,继续添加`Link`到了`Header`允许用户导航到`Login`页. 

<Instruction>

打开`Header.js`并更新`render`看起来如下: 

```js(path=".../hackernews-react-apollo/src/components/Header.js")
render() {
  const authToken = localStorage.getItem(AUTH_TOKEN)
  return (
    <div className="flex pa1 justify-between nowrap orange">
      <div className="flex flex-fixed black">
        <div className="fw7 mr1">Hacker News</div>
        <Link to="/" className="ml1 no-underline black">
          new
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
      <div className="flex flex-fixed">
        {authToken ? (
          <div
            className="ml1 pointer black"
            onClick={() => {
              localStorage.removeItem(AUTH_TOKEN)
              this.props.history.push(`/`)
            }}
          >
            logout
          </div>
        ) : (
          <Link to="/login" className="ml1 no-underline black">
            login
          </Link>
        )}
      </div>
    </div>
  )
}
```

</Instruction>

你首先检索`authToken`来自本地存储. 如果`authToken`不可用,**提交**- 按钮不会再渲染. 这样,您可以确保只有经过身份验证的用户才能创建新链接. 

你还在右侧添加了第二个按钮`Header`用户可以用来登录和注销. 

<Instruction>

最后,您需要从中导入密钥定义`constants.js`在`Header.js`. 将以下语句添加到文件顶部: 

```js(path=".../hackernews-react-apollo/src/components/Header.js")
import { AUTH_TOKEN } from '../constants'
```

</Instruction>

以下是ready组件的外观: 

![](http://imgur.com/tBxMVtb.png)

完美,你现在都准备好实现身份验证功能. 

### 使用身份验证突变

`signup`和`login`是两个常规的GraphQL突变,你可以像使用它一样使用`createLink`从以前突变. 

<Instruction>

打开`Login.js`并将以下两个定义添加到文件的顶部: 

```js(path=".../hackernews-react-apollo/src/components/Login.js")
const SIGNUP_MUTATION = gql`
  mutation SignupMutation($email: String!, $password: String!, $name: String!) {
    signup(email: $email, password: $password, name: $name) {
      token
    }
  }
`

const LOGIN_MUTATION = gql`
  mutation LoginMutation($email: String!, $password: String!) {
    login(email: $email, password: $password) {
      token
    }
  }
`
```

</Instruction>

这两种突变看起来与您之前看到的突变非常相似. 他们采取了一些论点并返回`token`您可以附加到后续请求以验证用户 (即表明发出了请求) *代表*那个用户) . 你会学到🔜怎么做. 

<Instruction>

也替换`flex mt3`班级名称`div`元素如下: 

```js{2-12}(path=".../hackernews-react-apollo/src/components/Login.js")
<div className="flex mt3">
  <Mutation
    mutation={login ? LOGIN_MUTATION : SIGNUP_MUTATION}
    variables={{ email, password, name }}
    onCompleted={data => this._confirm(data)}
  >
    {mutation => (
      <div className="pointer mr2 button" onClick={mutation}>
        {login ? 'login' : 'create account'}
      </div>
    )}
  </Mutation>
  <div
    className="pointer button"
    onClick={() => this.setState({ login: !login })}
  >
    {login ? 'need to create an account?' : 'already have an account?'}
  </div>
</div>
```

</Instruction>

在我们仔细研究之前`<Mutation />`组件实现,继续并添加所需的导入. 

<Instruction>

还在`Login.js`,将以下语句添加到文件的顶部: 

```js(path=".../hackernews-react-apollo/src/components/Login.js")
import { Mutation } from 'react-apollo'
import gql from 'graphql-tag'
```

</Instruction>

现在,让我们了解一下发生了什么`<Mutation />`你刚刚添加的组件. 

代码非常简单. 如果用户想要登录,那么你正在呼叫`loginMutation`,否则你正在使用`signupMutation`因此会在div上触发突变`onClick`事件. GraphQL突变收到`email`,`password`和`name`传递给它的状态值`variables`支柱. 最后,在变异完成后,我们打电话`_confirm`函数作为参数传递,返回变异`data`. 

好吧,剩下要做的就是实施`_confirm`功能!

<Instruction>

打开`Login.js`并更新`_confirm`如下: 

```js(path=".../hackernews-react-apollo/src/components/Login.js")
_confirm = async data => {
  const { token } = this.state.login ? data.login : data.signup
  this._saveUserData(token)
  this.props.history.push(`/`)
}
```

</Instruction>

执行突变后,您将存储返回的`token`在`localStorage`并导航回根路由. 

> **注意**: 变异返回`data`依赖于GraphQL变异定义,这就是我们需要得到的原因`token`取决于触发的突变. 

您现在可以通过提供a来创建帐户`name`,`email`和`password`. 一旦你这样做了,**提交**- 按钮将再次呈现: 

![](https://imgur.com/z4KILTw.png)

如果您之前没有这样做,请继续测试登录功能. 跑`yarn start`并打开`http://localhost:3000/login`. 然后单击**需要创建一个帐户?**- 按钮并为您正在创建的用户提供一些用户数据. 最后,点击**创建帐号**- 按钮. 如果一切顺利,应用程序将导航回根路径并创建用户. 您可以通过发送来验证新用户是否在那里`users`查询中**开发**游乐场里**数据库**项目. 

### 使用身份验证令牌配置Apollo

既然用户能够登录并获得对GraphQL服务器进行身份验证的令牌,您实际上需要确保令牌附加到发送到API的所有请求. 

由于所有API请求实际上都是由创建和发送的`ApolloClient`在您的应用中,您需要确保它知道用户的令牌!幸运的是,Apollo提供了一种使用概念验证所有请求的好方法[中间件](http://dev.apollodata.com/react/auth.html#Header),实施为[阿波罗链接](https://github.com/apollographql/apollo-link). 

首先,您需要向应用程序添加所需的依赖项. 打开终端,导航到项目目录并键入: 

<Instruction>

```bash(path=".../hackernews-react-apollo")
yarn add apollo-link-context
```

</Instruction>

让我们看一下运行中的身份验证链接!

<Instruction>

打开`index.js`并输入以下代码*之间*创造了`httpLink`和实例化`ApolloClient`: 

```js(path=".../hackernews-react-apollo/src/index.js")
const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem(AUTH_TOKEN)
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : ''
    }
  }
})
```

</Instruction>

<Instruction>

在继续之前,您需要导入Apollo依赖项. 将以下内容添加到顶部`index.js`: 

```js(path=".../hackernews-react-apollo/src/components/CreateLink.js")
import { setContext } from 'apollo-link-context'
```

</Instruction>

每次都会调用此中间件`ApolloClient`向服务器发送请求. Apollo Links允许创建`middlewares`允许您在将请求发送到服务器之前修改请求. 

让我们看看它在我们的代码中是如何工作的,首先,我们得到了身份验证`token`从`local storage`如果它存在,那之后我们返回`headers`到了`context`所以httpLink可以读取它们. 

> **注意**: 您可以阅读有关Apollo身份验证的更多信息[这里](https://www.apollographql.com/docs/react/recipes/authentication.html). 

<Instruction>

现在你还需要确认`ApolloClient`使用正确的链接实例化 - 更新构造函数调用,如下所示: 

```js{2}(path=".../hackernews-react-apollo/src/index.js")
const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache()
})
```

</Instruction>

<Instruction>

然后直接导入从中检索令牌所需的密钥`localStorage`在同一个文件的顶部: 

```js(path=".../hackernews-react-apollo/src/index.js")
import { AUTH_TOKEN } from './constants'
```

</Instruction>

就是这样 - 现在所有的API请求都将通过身份验证`token`是可用的. 

### 需要在服务器端进行身份验证

您在本章中做的最后一件事是确保只有经过身份验证的用户才能这样做`post`新链接. 而且,每一个`Link`这是由一个人创造的`post`突变应该自动设置`User`谁发送了它的请求`postedBy`领域. 

要实现此功能,这次您还需要在服务器端进行一些小的更改. 

<Instruction>

打开`/server/src/resolvers/Mutation.js`并调整`post`解析器看起来如下: 

```js(path=".../hackernews-react-apollo/server/src/resolvers/Mutation.js")
function post(parent, { url, description }, ctx, info) {
  const userId = getUserId(ctx)
  return ctx.db.mutation.createLink(
    { data: { url, description, postedBy: { connect: { id: userId } } } },
    info,
  )
}
```

</Instruction>

通过这个改变,你正在提取`userId`来自`Authorization`请求的标头并直接使用它[`connect`](https://www.prismagraphql.com/docs/reference/prisma-api/mutations-ol0yuoz6go#nested-mutations)它与`Link`那是创造的. 注意`getUserId`将[抛出一个错误](https://github.com/howtographql/react-apollo/blob/master/server/src/utils.js#L12)如果未提供字段或无法提取有效标记. 

> **注意**: 停止服务器并再次运行它`yarn dev`应用所做的更改. 
