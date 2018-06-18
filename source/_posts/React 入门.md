---
title: React 入门
date: 2018-06-17 16:59:12
tags: 
- JavaScript 
- React
categories:
- 笔记📒
---
学习 React 已经是很早就有的想法了, 因为之前的工作一直接触的都是 `Vue.js` 并且知道它俩的有异曲同工之妙, 但在写法上还是有蛮大区别的. 在提到 `Vue` 的地方也总是伴随着 `React` 像是麦当劳和肯德基一样(加上现在的前端都要怀揣一两个框架傍身🤷‍), 其实兴趣还挺大的, 但是在工作和准备跳槽等琐事下一直耽误. 到最近才有一些时间跟着文档写一点小 demo 入门 [React 16.2.0](https://reactjs.org/docs/try-react.html) , 顺便记录一下学习过程.

#### 安装

官网上有几种使用方式, 我就直接跳到常用的脚手架命令 `create-react-app`, 需要 `node >= 6.0`

```shell
# 全局安装
npm install -g creat-react-app

# 使用 create-react-app 并创建 react-start 文件夹
create-react-app react-start

# 进入 react-start
cd react-start

# 启用项目
npm start
```

如果你使用的 `yarn` 安装则替换为一下命令:

```shell
# 全局安装
yarn global add create-react-app --prefix /usr/local

# 启用换成相应的 yarn
yarn start
```

等待安装完成之后, 就成功启动了第一个 react 项目. 看一眼项目结构, 呃... 这么简洁的吗?

#### 目录结构

默认的结构非常简洁, 简单到怀疑是不是自己哪里搞错了...

```shell
react-srart
  - node-modules/
  - public/
  - src/
  - .gitignore
  - package.json
  - README.md
  - yarn.lock
```

可以发现没有看到 `webpack` 的配置, 查了一波原来是被隐藏掉了. 打开 `package.json`:

```json
"script": {
  "eject": "react-scripts eject"
}
```

我们运行这条命令 `yarn eject`, 在终端就可以看到以下输出:

```shell
➜  react-start git:(master) ✗ yarn eject
yarn run v1.7.0
$ react-scripts eject
? Are you sure you want to eject? This action is permanent. Yes
Ejecting...

# config 和 script 两个文件夹
Copying files into /Users/cara/workspace/project/react-staff/react-start
  Adding /config/env.js to the project
  Adding /config/paths.js to the project
  Adding /config/polyfills.js to the project
  Adding /config/webpack.config.dev.js to the project
  Adding /config/webpack.config.prod.js to the project
  Adding /config/webpackDevServer.config.js to the project
  Adding /config/jest/cssTransform.js to the project
  Adding /config/jest/fileTransform.js to the project
  Adding /scripts/build.js to the project
  Adding /scripts/start.js to the project
  Adding /scripts/test.js to the project

# 添加了依赖项 (太多就省略了)
Updating the dependencies
...

# 替换 package.json 里的命令
Updating the scripts
  Replacing "react-scripts start" with "node scripts/start.js"
  Replacing "react-scripts build" with "node scripts/build.js"
  Replacing "react-scripts test" with "node scripts/test.js"

Configuring package.json
  Adding Jest configuration
  Adding Babel preset
  Adding ESLint configuration

Ejected successfully!

Please consider sharing why you ejected in this survey:
  http://goo.gl/forms/Bi6CZjk1EqsdelXk1

✨  Done in 2.20s.
```

接着项目目录中就多了两个文件夹, `scripts` 和 `config` 又看到了我们熟悉的 `webpack` 的配置, 同时 `package.json` 也发生一些变化.

#### React 基本

知道了如何创建新的项目, 接下来该了解一下 `React` 的基本 API 和 生命周期了. 在 React 的世界里一切皆函数, 使用的 `JSX` 一种 JavaScript 的扩展语法. 下面来看一下 JSX 的基本用法

- JSX 中嵌入表达式

```js
const formatString = (name, type) => {
  return name + ' frist ' + type + 'appliction!'
}

const element = (
  <h1>
    Hello, { formatString('cara', 'React') }
  </h1>
)

ReactDOM.render(
  element,
  document.getElementById('root')
)
```

- JSX 作为表达式

```js
const createElement = (user) => {
  if (user) {
    return <h1>Hello, {formatString('cara', 'React')}</h1>
  } else {
    return <h1>Hello, Stranger</h1>
  }
}
```

- JSX 在指定属性时, 必须使用驼峰命名, 表达式通过花括号计算

```js
const image = <img src={user.avatarImg} />
```

- JSX 也可以包含子元素

> React 在多次渲染同一个组件时, 不会这个销毁再重新渲染, 而是只更新需要更新的部分.

##### Components 和 Props

** 函数式组件和类组件 **

一个有效的函数式组件, 可以接收一个 props 并返回一个 React 元素. 因为它本身长的就很像函数, 所以称之为函数式组件

```js
// 函数式组件
const Welcome = (props) => {
  return <h1>Hello, {props.name}</h1>
}
```

也可以用 ES6 中的 Class 来定义一个组件

```js
// 类组件
class Welcome extend React.Component {
  render () {
    return <h1>Hello, {this.props.name}</h1>
  }
}
```

渲染的一个组件:

```js
const element = <Welcome name="cara" />

ReactDOM.render(
  element,
  document.getElementById('root')
)
// 显示为 Hello, cara
```

通过 `this.props` 就能获取到从组件外层传入的自定义属性, 跟 Vue 组件内部的 `props` 类似.

> 注意: 组件的名称都以大写字母开头, 属性名使用驼峰命名; 组件中跟 Vue 一样都需要一个根节点.

** 提取组件 **
react 中提倡把一个组件拆分成多个小组件便于复用, 尤其是 UI 组件. 我理解的是因为 React 一切皆函数, 具体到 `button` 这种小的 UI 组件, 每次都重写一遍就很没意义, 所以尽量拆小一点以便复用. 官网中以评论框为例, 这里就不再复述.

** Props 是只读的**
在 React 中有一条非常严格的规则: 所有 React 的组件都必须是纯函数, 不得改变自身组件的 props. (纯函数是指: 不会改变参数的输入, 对于相同的输入始终可以得到相同的结果) 看到官网这么说我也很疑惑 UI 组件不可能都是纯的静态组件啊, 肯定有根据状态动态改变视图的, 或者做一些过滤和格式化的操作. 于是到了状态和生命周期

> react 写 html 标签在 vscode 中使用 emmet 自动补全需要如下配置:

```json
"emmet.includeLanguages": {
    "javascript": "javascriptreact"
}
```

#### State 和生命周期

state 和 props 类似都在组件内部由组件自己维护, 可以说是组件自身的一个局部状态吧. 举例说明:

我们需要一个在内部自动更新的组件, 在内部维护自己的状态

```js
// 从 React.Component 继承一个 Clack 类
class Clack extends React.Component {
  // 始终使用 props 调用构造函数
  // 初始化 state
  constructor (props) {
    super(props)
    // 类似于 Vue 中的 data
    this.state = {
      date: new Date()
    }
  }
  render () {
    return (
      <div>
        <div>State Component</div>
        <div>Time is {this.state.date.toLocaleTimeString()}.</div>
      </div>
    )
  }
}
```

现在我们有了一个有着局部状态的类组件, 接下来让 Clack 设置自己的计时器, 两秒更新一次. 就会用到声明周期函数了, 组件在挂载时称为 `mounting`(挂载); 卸载时称为 `unmounting`(卸载)

```js
class Clack extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      date: new Date()
    }
  }

  // 组件第一次挂载到 DOM 时
  componentDidMount () {
    this.timer = setInterval(_ => this.ticker(), 2000)
  }

  // 组件被销毁时
  componentWillUnmount () {
    clearInterval(this.timer)
  }

  ticker () {
    // 更新 state
    this.setState({
      date: new Date()
    })
  }

  render () {
    return (
      <div>
        <div>State Component</div>
        <div>Time is {this.state.date.toLocaleTimeString()}.</div>
      </div>
    )
  }
}
```

于是在 Clack 组件第一次被挂载到 DOM 时, 通过 `componentDidMount` 生命周期钩子函数调用一个定时器, `setState` 改变组件状态来达到定时更新时间; 而在组件销毁时通过 `componentWillMount` 钩子来停止定时器.

** State 的注意事项 **

- 不要直接修改 state, 应该通过 setState().
- state 的更新可能是异步的, 因为 React 可能会将多个 setState() 合并为一次更新, 如果需要可以用函数作为参数, 如:

```js
this.setState((prevState, props) => {
  count: prevState.count + props.increment
})
```

- state 的更新可能会被合并. 两个独立状态分别更新, 前一个状态可能不会被更新但后一个状态一定会被更新.
- 数据向下流动, 一个组件的 state 可以向下传递作为子组件的 props, 也就是常说的单向数据流.

以上就是这次初步学习 React 的经历, 大部分跟 Vue 还是相似只是换了一种写法而已, 下次会继续深入学习其他部分.

created on 2018-06-18