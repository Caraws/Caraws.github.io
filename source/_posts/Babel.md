---
title: Babel生态
date: 2018-04-09 17:55:20
tags:
- Babel
categorles:
- 笔记📒
---
对于 babel 的使用, 一直停留在与 webpack 结合使用, 以及在 Vue 开发环境下脚手架又是开箱即用. 导致很多 babel 的包, 我都不清楚他们是干什么的. 比如 babel-register/ babel-runtime/ helpers/ 各种 presets 以及 transform-runtime 和 babel-polyfill 的区别, 所以总结一下.

### babel-cli
我理解为 babel 的 command 全家桶, 用于在命令行操作的. 里面包含了 `babel`/ `babel-external-helpers`/ `babel-node` 3个命令

- babel: 用于编译代码
- babel-external-helpers: 用于生成一些 halper 函数, 包含 babel 所有的 hepler 函数 (如: toArray 函数, jsx 转化函数). 这些函数实在 babel transform 的时候用, 都放在 `babel-helpers` 这个包中, 当这些 helpers 被用到就会被放置在生成代码的顶部. 但是当多个文件都用到了 helpers 函数就会产生冗余代码, 所以 babel 提供这个命令生成一个包含所有 helpers 的 js 文件用于直接引用.(然后可以通过 plugin 去检查全局时候存在这个模块, 存在就不定义)
- babel-node: 主要是实现在 node 中写代码和执行脚本的能力, 可以直接运行 ES6代码. 比如直接在 node 中写 jsx, 通过这个就可以执行. 但是要把它编译成可执行的脚本还需要 `babel-register`

安装
`npm install --global babel-cli`

用法
```shell
babel example.js

# 转码结果写入文件
babel example.js -o result.js

# 整个目录转码
babel src -d lib

# 生成 source map 文件
babel src -d lib -s
```

全局环境下 babel 无法支持不同版本的 babel, 所以安全的做法还是把 babel 装在项目中
```shell
npm install --save-dev babel-cli
```

package.json 改成:
```json
{
  "script": "babel src -d lib"
}
```

**babel-register**
它的特点就是实时编译, 不会输出文件, 用来改写 `require` 命令为它加上钩子. `require` 进来的文件就会被转码, 但是它不会转码当前文件中的代码.

安装
```js
npm install --save-dev babel-register
```

使用
```js
// 必须先加载 register
require('babel-register')
// 然后 register 就会对 test.js 文件转码
require('test.js')
```

> 由于 `babel-register` 是即时转码, 所以只适用于开发环境使用.

### babel-core
`babel-core` 可以说是 babel 最为核心的一个包, 可以把它看成一个编译器, babel 核心的 API 都在里面. 比如: transform 处理转码, 因为 ES6 的语法跟老语法不同, 所以先将我们的代码转换为 AST(抽象语法树), 然后分别做处理转化为 ES5. webpack 的 babel-loader 就是调用这些 API 完成转译的. [这里是详细的 API](https://babeljs.cn/docs/usage/api/)

> 这里需要注意的是: `babel-core` 仅关注 code transform, 也就是说它只做语法上的转换, 比如箭头函数. 所以并不是什么都能用 babel 来转换的, 如果涉及到新的 API 就需要你用 polyfill 来转译, 比如 `Promise`.

```js
// 整块引入
var babel = require('babel-core')
// 也可以选择某个 API 单独使用
import { transform } from 'babel-core'
```

主要的 API:

**babel.transform(code: string, options?: Object)**
将传入的 code 做转换, 返回值为一个对象, 参数为: 生成的对象, source map 和 AST

```js
var result = babel.transform("code();", options);
result.code
result.map
result.ast
```

**babel.transformFile(filename: string, options?: Object, callback: Function)**
异步转译文件中的所有内容

```js
babel.transformFile('filename.js', options, function (err, result) {
  result // { code, map, ast }
})
```

**babel.transformFileSync(filename: string, options?: Object)**
`babel.transformFile` 的同步版本, 返回值为 `filename` 文件中转译后的代码

```js
babel.transformFileSync(filename, options) // { code, map, ast }
```

**babel.transformFromAst(ast: Object, code?: string, options?: Object)**
反转译, 给一个 AST 转为 code

```js
const code = 'if (true) return'
const ast = babylon.parse(code, { allowReturnOutsideFunction: true })
const { code, map, ast } = babel.transformFormAst(ast, code, options)
```

### babel-runtime
[`babel-runtime`](https://babeljs.cn/docs/plugins/transform-runtime/) 这个包其实就是把 core-js 和 regenerator 组合起来供使用, 它和 `babel-polyfill` 的都是为了模拟 ES6 环境. 之前提到的 `babel-core` 只对语法进行转换, 但不支持 Promise, Set, Map, array.reduce, Array.form, genertor, async 这些新 API 的编译, 所以才会用到这两个东西.

**core-js**: 主要实现了 Promise, Symbols, ES7提案等等的 polyfill, 包含了大部分的 JavaScript 最新标准的垫片.
```js
// 需要单个引用后再使用
require('core-js/array/reduce')
```

**regenerator**: 主要实现 generator/yeild, async/await (不知道为啥 core-js 不把这两个一起实现了...)

**helpers**: `babel-runtime` 里也有 helpers, 它里面的 helpers 相当于之前提到的 `babel-external-helpers` 生成的 helpers.js, 只不过把每个 helpers 函数都单独放到一个文件夹里而已. 这样配合 transform 的时候, 需要用到 helpers 函数就可以直接从 `babel-runtime` 中引用了.
```js
const _asyncToGenerator2 = require('babel-runtime/helpers/asyncToGenerator')
```

区别: 

- [babel-polyfill](https://babeljs.cn/docs/usage/polyfill): 是通过全局对象和内置对象的 prototype 上添加方法来达到目的, 所以一旦引入 `babel-polyfill` 就会污染全局环境.
- babel-runtime: `babel-runtime` 是一个模块, 所以它不会污染全局环境和内置对象的原型. 它的做法引入需要的 helper 函数(类似: `const Promise = require('babel-runtime/core-js/promise')` 来引入 Promise).

优缺点:

- babel-poly-fill: 引入一劳永逸, 但是污染环境.
- babel-runtime: 
    1. 手动引入不方便
    2. 直接在代码中引入 helper 函数意味着不能共享, 最终打包出来会有很多冗余代码(引入的都是全量的 polyfill). 
    
所以要配合 `babel-plugin-transform-runtime` (很多地方都是把 `babel-runtime` 和 `babel-plugin-transform-runtime` 统称为 transform-runtime, 因为它俩得合在一起才好用) 来达到按需引入 helper 避免重复打包和手动引入的痛苦, 它主要是做一层映射: 将 `babel-runtime` 内引用到的 core-js 或 regenerator.js 映射到具体对应的 helper. 

> 注意: `babel-runtime` 无法转码实例方法, 即内置对象原型上的方法, 只能通过 `babel-polyfill` 来转码. 比如:
```js
Array.prototype.find()
'hello'.includes('h')
```

另外, 关于为什么 `babel-runtime` 是 dependencies 依赖, 因为他只是集合了 polyfill 的一个 library, 对应需要的 polyfill 都是要引入项目中, 并跟项目代码一起打包的.

### babel-preset-env
它能根据当前的运行环境, 自动确定你需要的 plugins 和 polyfills, babel 的配置官方推荐是写到 `.babelrc` 文件中, 以 `Vue-cli` 的 babel 配置为例.[详细参数设置及说明](https://babeljs.cn/docs/plugins/preset-env/)

```json
{
  "presets": [
    ["env", 
      { 
        "modules": false // 设置 ES6 模块转译的模式, 默认是 commonjs
      }
    ],
    // 需要支持到哪个阶段的 JavaScript 版本
    "stage-2"
  ],
  // 需要的插件
  "plugins": [
    "transform-runtime", // 虽然这里没有写 babel-runtime, 但是 transform-runtime 依赖于它, 所以还是要安装
    "transform-vue-jsx"
  ],
  // 编译过程是否保留注释
  "comments": false,
  "env": {
    "test": {
      "presets": ["env", "stage-2"],
      "plugins": [ "istanbul"]
    }
  }
}
```

本来 `babel-preset-env` 中有一个 useBuiltIns 选项(默认值是 false), 就是它实现的根据运行环境并判断需要什么 polyfill, 达到按需引入而不是整个引入, 对于 `import 'babel-polyfill'` 就很棒了. 但是可以看到 vue-template 并没有使用这个选项, 这是因为   <a href='javascript:' title='我消不掉这个超链接😑'>babel-preset-env@1.x</a> 的版本没有办法很好消除未使用的 polyfill, 在2.x 版本下可以用 `usebuiltIns: 'usage'`达到目的.

### 其他
babel 还有很多相关的东西没说到, 只挑了几个最重要的来说, 具体的 helpers/ plugins 都可以到[官方仓库](https://github.com/babel/babel/tree/master/packages)查看

- [badylon](https://github.com/babel/babel/tree/master/packages/babylon): 词法解析器
- [babel-traverse](https://github.com/babel/babel/tree/master/packages/babel-traverse): 用于遍历, 维护整个 AST 的状态
- [babel-generator](https://github.com/babel/babel/tree/master/packages/babel-generator): 根据 AST 生成代码

#### babel 的工作流程

输入需要转码的代码 -> badylon 解析 -> 得到 AST -> 
babel-traverse 遍历转译 -> 得到新的 AST -> 
最后 babel-generator 根据新的 AST 生成代码

### 总结
整个来说我觉得还是 `babel-runtime` 和 `babel-polyfill` 比较难区分, 其实两者的核心都在于 [core-js](https://github.com/zloirock/core-js#basic) 只是各有优缺点而已, 下面会再简要总结一下区别. 其他的话只需要知道每个东西是干啥的就行了.

- babel-polyfill: 污染环境, 支持实例方法(如果只想引入一些特定的polyfill, 那就去 [core-js](https://github.com/zloirock/core-js#basic) 中找相应的方法自己手动 require 进来); 
- runtime: 按需引用,不支持实例方法.

Created on 2018-4-11 by Cara