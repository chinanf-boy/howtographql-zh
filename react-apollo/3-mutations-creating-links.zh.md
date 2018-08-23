---
title: "Mutations: Creating Links"
pageTitle: "GraphQL Mutations with React & Apollo Tutorial"
description: "Learn how you can use GraphQL mutations with Apollo Client. Use Apollo's `<Mutation />` component to define and send mutations."
question: Which of the following statements is true?
answers: ["Only queries can be wrapped with the 'graphql' higher-order component", "'<Mutation />' component allow variables, optimisticResponse, refetchQueries, and update as props", "When wrapping a component with a mutation using 'graphql', Apollo only injects the mutation function into the render prop function", "GraphQL mutations never take any arguments"]
correctAnswer: 1
videoId: ""
duration: 0		
videoAuthor: ""
---
在本节中,您将学习如何使用Apollo发送突变. 它实际上与发送查询没有什么不同,并遵循前面提到的相同的三个步骤,最后两个步骤中的未成年人 (但逻辑) 差异: 

1.  使用. 将变异写为JavaScript常量`gql`解析器功能
2.  使用`<Mutation />`组件传递GraphQL变异和变量 (如果需要) 作为道具
3.  使用注入组件的变异函数`render prop function`

### 准备React组件

像以前一样,让我们​​从编写React组件开始,用户可以在其中添加新链接. 

<Instruction>

在中创建一个新文件`src/components`目录并调用它`CreateLink.js`. 然后将以下代码粘贴到其中: 

```js(path=".../hackernews-react-apollo/src/components/CreateLink.js")
import React, { Component } from 'react'

class CreateLink extends Component {
  state = {
    description: '',
    url: '',
  }

  render() {
    const { description, url } = this.state
    return (
      <div>
        <div className="flex flex-column mt3">
          <input
            className="mb2"
            value={description}
            onChange={e => this.setState({ description: e.target.value })}
            type="text"
            placeholder="A description for the link"
          />
          <input
            className="mb2"
            value={url}
            onChange={e => this.setState({ url: e.target.value })}
            type="text"
            placeholder="The URL for the link"
          />
        </div>
        <button onClick={`... you'll implement this 🔜`}>Submit</button>
      </div>
    )
  }
}

export default CreateLink
```

</Instruction>

这是具有两个的React组件的标准设置`input`用户可以提供的字段`url`和`description`的`Link`他们想要创造. 输入这些字段的数据存储在组件中`state`并将在发送变异时使用. 

### 写出变异

但是你现在怎么能把变异发送到服务器呢?让我们按照之前的三个步骤进行操作. 

首先,您需要在JavaScript代码中定义变异并使用`graphql`容器. 您将以与之前的查询类似的方式执行此操作. 

<Instruction>

在`CreateLink.js`,将以下语句添加到文件的顶部: 

```js(path=".../hackernews-react-apollo/src/components/CreateLink.js")
const POST_MUTATION = gql`
  mutation PostMutation($description: String!, $url: String!) {
    post(description: $description, url: $url) {
      id
      createdAt
      url
      description
    }
  }
`
```

</Instruction>

<Instruction>

也替换当前`button`以下内容: 

```js(path=".../hackernews-react-apollo/src/components/CreateLink.js")
<Mutation mutation={POST_MUTATION} variables={{ description, url }}>
  {() => (
    <button onClick={`... you'll implement this 🔜`}>
      Submit
    </button>
  )}
</Mutation>
```

</Instruction>

让我们再仔细看看,了解发生了什么: 

1.  首先创建一个名为的JavaScript常量`POST_MUTATION`存储突变. 
2.  现在,你包装了`button`元素为`render prop function`结果`<Mutation />`组件传递`POST_MUTATION`作为道具. 
3.  最后你通过*描述*和*网址*陈述为`variables`支柱. 

<Instruction>

在继续之前,您需要导入Apollo依赖项. 将以下内容添加到顶部`CreateLink.js`: 

```js(path=".../hackernews-react-apollo/src/components/CreateLink.js")
import { Mutation } from 'react-apollo'
import gql from 'graphql-tag'
```

</Instruction>

让我们看看行动中的变异!

<Instruction>

还在`CreateLink.js`,替换`<Mutation />`组件如下: 

```js(path=".../hackernews-react-apollo/src/components/CreateLink.js")
<Mutation mutation={POST_MUTATION} variables={{ description, url }}>
  {postMutation => <button onClick={postMutation}>Submit</button>}
</Mutation>
```

</Instruction>

正如所承诺的,你需要做的就是调用Apollo注入的功能`<Mutation />`组件`render prop function`在onClick按钮的事件内. 

<Instruction>

来吧看看突变是否有效. 为了能够测试代码,请打开`App.js`并改变`render`看起来如下: 

```js(path=".../hackernews-react-apollo/src/components/App.js")
render() {
  return <CreateLink />
}
```

</Instruction>

<Instruction>

接下来,导入`CreateLink`通过在顶部添加以下语句来组件`App.js`: 

```js(path=".../hackernews-react-apollo/src/components/App.js")
import CreateLink from './CreateLink'
```

</Instruction>

现在,跑`yarn start`,您将看到以下屏幕: 

![](http://imgur.com/AJNlEfj.png)

两个输入字段和一个**提交**- 按钮 - 不是很漂亮但功能齐全. 

在字段中输入一些数据,例如: 

-   **描述**: `The best learning resource for GraphQL`
-   **网址**: `www.howtographql.com`

然后单击**提交**- 按钮. 您不会在UI中获得任何视觉反馈,但让我们通过检查Playground中的当前链接列表来查看查询是否真正有效. 

您可以通过导航到再次打开游乐场`http://localhost:4000`在您的浏览器中. 然后发送以下查询: 

```graphql
# Try to write your query here
{
  feed {
    links {
      description
      url
    }
  }
}
```

您将看到以下服务器响应: 

```js(nocopy)
{
  "data": {
    "feed": {
      "links": [
        // ...
        {
          "description": "The best learning resource for GraphQL",
          "url": "www.howtographql.com"
        }
      ]
    }
  }
}
```

真棒!突变起作用,很棒!💪
