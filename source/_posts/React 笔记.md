---
title: React 笔记
date: 2018-06-17 16:59:12
tags: 
- JavaScript 
- React
categories:
- 笔记📒
---
学习 React 已经是很早就有的想法了, 因为之前的工作一直接触的都是 `Vue.js` 并且知道它俩的有异曲同工之妙, 但在写法上还是有蛮大区别的. 在提到 `Vue` 的地方也总是伴随着 `React` 像是麦当劳和肯德基一样(加上现在的前端都要怀揣一两个框架傍身🤷‍), 其实兴趣还挺大的, 但是在工作和准备跳槽等琐事下一直耽误. 到最近才有一些时间跟着文档写一点小 demo 入门 [React 16.2.0](https://reactjs.org/docs/try-react.html) , 顺便记录一下学习过程.

#### 安装

直接使用 `create-react-app`, 这个过程想必大家都很熟悉了就不再说. 等待安装完成之后, 就成功启动了第一个 react 项目. 看一眼项目结构, 呃... 这么简洁的吗?

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
在 React 中有一条非常严格的规则: 所有 React 的组件都必须是纯函数, 不得改变自身组件的 props. (纯函数是指: 不会改变参数的输入, 对于相同的输入始终可以得到相同的结果) 看到官网这么说我也很疑惑 UI 组件不可能都是纯的静态组件啊, 肯定有根据状态动态改变视图的, 或者做一些过滤和格式化的操作. 于是来到状态和生命周期

> react 写 html 标签在 vscode 中使用 emmet 自动补全需要如下配置:

```json
"emmet.includeLanguages": {
  "javascript": "javascriptreact"
}
```

##### State 和生命周期

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

现在我们有了一个有着局部状态的类组件, 接下来让 Clack 设置自己的计时器, 两秒更新一次.

```js
class Clack extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      date: new Date()
    }
  }

  // 组件第一次挂载到 DOM 时(钩子)
  componentDidMount () {
    this.timer = setInterval(_ => this.ticker(), 2000)
  }

  // 组件被销毁时(钩子)
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

于是在 Clack 组件第一次被挂载到 DOM 时, 通过 `componentDidMount` 生命周期钩子调用一个定时器, `setState` 改变组件状态来达到定时更新时间; 而在组件销毁时通过 `componentWillMount` 钩子来停止定时器.

**组件的生命周期**
组件的生命周期分为三大类: `Mounting`(装载时); `updating`(更新); `unmounting`(卸载时). 跟 Vue 一样你可以在钩子函数里运行一些特定的代码.

1. Mounting(装载): 当组件实例创建且插入 DOM 时, 下面的方法将会被调用
  - constructor()
    初始化 props/ state 或者绑定事件处理程序

  - componentWillMount()

  - render()
    通常用来返回 React 元素(数字/字符/portals/null: 不渲染/布尔值:不渲染任何东西) 且应该是纯函数, 注意当 `shouldComponentUpdate()` 返回 false 时, render 不会被调用.

  - componentDidMount()
    在组件装载后立即调用, 等同于 Vue 的 mounted(). 在这个函数中使用 `setState()` 会触发两次 render(但不会被用户感知).

2. Updating(更新): 当 props 和 state 发生改变, 重新渲染组件会触发以下钩子

  - componentWillReceiveProps(nextProps)
    `componentWillReceiveProps(nextProps)` 在响应 props 变化之前被调用. 有个注意点是有可能 props 没有更新 React 也调用了这个函数, 所以如果你只关心变化需要自己比较当前值和下一个值.

  - shouldComponentUpdate(nextProps, nextState)
    组件是否受 props 和 state 变化的影响, 默认返回 true. 首次渲染或使用 `forceUpdate()` 时不调用此方法;  如果 `shouldComponentUpdate()` 返回 false, 那么 `componentWillUpdate()`, `render()` 和 `componentDidUpdate()` 将不会调用. 所以你可以手动返回 false 告诉 React 这次不必更新, 另外不要在这里进行深度检查(如: 深拷贝) 会非常消耗性能.

  - componentWillUpdate(nextProps, nextState)
    该函数在第一次渲染时不会被调用, 会在收到 `nextProps` 或 `nextState` 之后在更新渲染之前被立即调用. 需要主要注意的是这里不能使用 `setState()`, 如果需要用 state 响应 props 的变化, 改用 `componentWillReceiveProps()`.

  - render()

  - componentDidUpdate(prevProps, prevState)
    第一次渲染时不会调用, 在更新发生后立即被调用

3. Unmounting(卸载): 组件从 DOM 上删除

  - componentWillUnmount()
    在组件被卸载或者销毁之前调用, 在这里通常用来取消一些订阅.

个人觉得 React 的生命周期钩子名字要比 Vue 的好理解一点, 起码一看名字就知道是啥.

**其他 API**

- setState(): 只是一个请求而不是立即命令来更新组件, 所以并不总是立即更新. 如果需要在 `setState()` 之后立即读取 `this.state`, 需要使用 `componentDidUpdate` 或者 `setState(updater, callback)`.(setState() 的第一个参数可以是对象, 也可以是函数)

  ```js
  // 参数为对象
  this.setState({count: 2})

  // 第一个参数为函数会保证 prevState, props 是最新的, 第二个参数是可选的 callback
  // callback 会在 setState 完成后执行并重新渲染组件
  this.setState((prevState, props) => {
    return { count: prevState.count + props.increment }
  })

  // 使用到 callback 的话, 建议使用 componentDidUpdate()
  ```

- defaultProps: 为类设置默认的 props.
  使用下面的组件时不传入 `props.color`, 他就会被设置为 `'blue'`. 如果被手动设置为 null 那它会保持为 null

  ```js
    class Ball extends React.Component {
      // ...
    }

    Ball.defaultProps = {
      color: 'blue'
    }

    // 在其他地方使用时, color 将保持为 null
    render () {
      return <Ball color={null} />
    }
  ```

**state 的注意事项**

- state 是组件内部的状态, 如果不在 `render()` 中使用那它就不应该是 `state`. 永远不要直接改变 `state`
- 数据向下流动, 一个组件的 state 可以向下传递作为子组件的 props, 也就是常说的单向数据流.

created on 2018-06-18