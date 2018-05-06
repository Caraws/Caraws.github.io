---
title: Promise 分析及实现
date: 2018-01-04 19:07:22
tags:
- JavaScript
categories:
- 笔记📒
---
Promise 是 ES6中中收录的异步操作封装, 通常在回调/ 事件/ 消息等异步操作中有显著的优势, 让我们在更方便的操作异步也让代码更加清晰.包括 ES7中的 Async/Await 也是对异步操作的封装, 不过 Async 更像是 Generator 的语法糖.

### 基础实现
学习[剖析 Promise 之基础篇](https://tech.meituan.com/promise-insight.html)用一个最常见的应用来剖析 `Promise`, 通过异步获取用户 Id, 然后作一些处理. 平常我们最常用的是回调的方式来处理, 下面用 `Promise` 的方式来处理.
```javascript
const getUserId = _ => {
    new Promise((resolve, reject) => {
        axios.get('http://example.com/api', params: param).then(res => {
            resolve(JSON.parse(res).id)
        })
    })
}
getUserId().then(id => {
    console.log(id)
    // do something...
})
```
getUserId 函数返回一个 `Promise`, 在他的 `then` 方法中放入异步操作成功之后的回调函数. 这种方式明显比我们之前常用的回调函数更加方便而且易读, 也更加容易避免 callback hell.

那么满足这样的场景的`Promise`是怎样实现的呢, 下面我们可以简单的实现一下最基础的`Promise`:
```javascript
function Promise (fn) {
    let value = null
    let deferreds = []
    
    // 4. 存入异步成功需要的回调函数
    // 此时指向两个回调函数 id => {}
    this.then = function (onFulfilled) {
        deferreds.push(onFulfilled)
        return this
    }
    
    // 3. 执行 deferreds 队列中的回调函数
    // 此时 value 为123
    function resolve (value) {
        deferreds.map(deferred => {
            deferred(value)
        })
    }
    
    // 1. 执行创建 Promise 实例时传入的函数
    // 并传入 resolve 供在适当时触发回调
    fn(resolve)
}

const getUserId = _ => {
    return new Promise(resolve => {
        // 2. 执行 resolve
        resolve(123)
    })
}
getUserId().then( id => {
    console.log('this is callback!')
    console.log(id) // 123
}).then(id => {
    console.log('this is second callback!')
    console.log(id) // 123
})
```
 - 调用 `then` 方法将回调函数存入 deferreds 队列.
 - 创建 `Promise` 实例时传入函数和 `resolve`, `resolve` 用于在适当的时间触发回调函数.
 - 真正执行回调函数的是 `deferreds` 队列中的元素.
 - `resolve` 函数接受一个参数, 用于回调函数使用, 即异步操作的返回结果.

可能大家已经发现, 以上代码并不能真正执行到回调函数.根据上面标注的序号就是代码的执行顺序, 这是因为现在还是同步函数, `Promise` 函数中的 `resolve` 函数会先于 `this.then` 函数执行,此时 `deferreds` 队列中还是空的, 以至于后面的回调函数也无法执行. 所以我们要保证回调以异步的方式执行, 以保证执行顺序. 可以通过 `setTimeout` 将 `resolve` 中的回调函数放在执行栈的末尾.
```javascript
function resolve (value) {
    // 将执行的回调的逻辑放入执行栈末尾
    setTimeout(function () {
        deferreds.map(deferred => {
            deferred(value)
        })    
    }, 0)
}
```
现在就可以看到, then 中的回调函数能够正常执行了.

#### 引入状态
现在我们引入规范 `Promises/A+` 中所说的 States, 它有三个互斥的状态: pending/ fulfilled/ rejected.

现在我们来改进下代码:
```javascript
function Promise (fn) {
    let value = null
    let deferreds = []
    let state = 'pending'
    
    this.then = function (onFulfilled) {
        if (state === 'pending') {
            deferreds.push(onFulfilled)
            return this
        }
        onFulfilled(value)
        return this
    }
    
    function resolve (newValue) {
        value = newValue
        state = 'fulfilled'
        setTimeout(_ => {
            deferreds.map(deferred => {
                deferred(value)
            })
        }, 0)
    }
    fn(resolve)
}
```

### 串行 Promise
串行 `Promise` 是指当 promise 达到 fuifilled 状态之后, 再进行下一个 promise. 比如上例中的我们拿到 userId 之后还需要用这个 userId 去获取用户的名称/ 住址/ 手机号等其他信息.

使用的伪代码类似这样:
```javascript
getUserId()
    .then(getUserInfoById)
    .then(userInfo => {
        // do something
    })
    
const getUserInfoById = id => {
    return new Promise(resolve => {
        axios.get('http://example.com/api', params: {
            id: 123
        }).then(response => {
            resolve(JSON.parse(response).info)
        })
    })
}
const getUserId = _ => {
    return new Promise(resolve => {
        resolve(123)
    })
}
```
串行的困难在于如何将前后的 `Promise` 衔接起来, 首先对 `then` 方法改造:
```javascript
this.then = function (onFulfilled) {
    // bridge promise
    return new Promise(resolve => {
        handle({
            onFulfilled: onFulfilled || null,
            resolve: resolve
        })
    })
}

let handle = deferred => {
    if (state === 'pending') {
        deferreds.push(deferred)
        return
    }
    let ret = deferred.onFulfilled(value)
    deferred.resolve(ret)
}
```
- `then` 方法中返回一个新创建的 Promise 实例作为返回值, 这是串行的基础, 由于返回类型一样所以依然支持链式.
- `then` 方法中的形参 `onFulfilled` 和新创建的 Promise 实例中的 `resolve` 均放入当前 promise 的 deferreds 队列中.
- `handle` 方法作为当前 promise 的内部方法, 较之前的 `then` 方法只增加了一行`deferred.resolve(ret)`.

在当前 promise 的异步成功之后执行 `handle` 方法时, 先执行 `onFulfilled` 方法, 然后将其返回值作为 `resolve` 方法的实参传入.

再改造 `resolve` 方法, 把代码整理一下:
```javascript
function Promise (fn) {
    let value = null
    let deferreds = []
    let state = 'pending'
    
    this.then = function (onFulfilled) {
        // 创建一个新的 Promise作为返回值
        // 将当前 promise 的回调函数和新创建的 resolve
        // 放入 deferreds 队列
        return new Promise (resolve => {
            handle({
                onFulfilled: onFulfilled || null
                resolve: resolve
            })
        })
    }
    const handle = deferred => {
        // 初始化状态时, 往 deferreds 队列添加
        if (state === 'pending') {
            deferreds.push(deferred)
            return
        }
        // 当前 promise 达到 'fulfilled' 状态之后
        // 先执行回调函数, 再将回调函数的返回值(ret)
        // 传递给 resolve 函数
        let ret = deferred.onFulfilled(value)
        deferred.resolve(ret)
    }
    
    const resolve = newValue => {
        if (newValue && (typeof newValue === 'object' || typeof newValue === 'function')) {
            let then = newValue.then
            if (typeof then === 'function') {
                then.call(newValue, resolve)
                return
            }
        }
        value = newValue
        state = 'fulfilled'
        setTimeout(_ => {
            deferreds.map(deferred => {
                handle(deferred)
            })
        }, 0)
    }
    // 执行 promise 实例中的回调函数
    fn(resolve)
}

getUserId()
    .then(getUserInfoById)
    .then(userInfo => {
        // do something
        console.log('this is second callback')
    })
```
现在 `resolve` 支持传入一个 promise 实例的参数了, 执行顺序如下:
1. `getUserId` 生成的 promise1, 进入 Promise 函数执行 `fn(resolve)` 由于`getUserId` 的内部是一个异步操作, 下一步会直接执行 `this.then(onFulfilled)`.
2. `this.then()` 返回一个新的 promise2 函数( 即: bridge promise)作为链式调用, promise2 重新实例化再执行 `fn(resolve)`, 则进入`handle({...})` 此时 handle 的参数为 getUserInfoById 和 resolve2, 接着被 push 到 `deferreds1` 队列中; 再接着执行下一个`this.then()` 生成 promise3, `deferreds2` 队列中 push 进第二个 then 的匿名函数(userInfo => {...}) 和 resolve3.
3. 执行`resolve(123)`, 进入 `resolve(newValue)` 执行`handle(deferred)` 此时 deferred 为 getUserInfoById 和 resolve2, 执行 handle 内部的 `deferred.onFulfilled(value)` 也就是 getUserInfoById 方法从而生成 promise4, 再到 `deferred.resolve(ret)` 这个时候 ret 就为 promise4, 传入 `resolve(newValue)` 执行 `then.call(promise4, resolve2)`.
4. 接着上一步进入 `this.then()` 生成 promise5(bridge promise), deferred4 压入 resolve2 和 resolve5; 在执行 getUserInfoById 中的 `resolve({name: 'cara'...})`, 进入 setTimeout 中的 `handle(deferred)`, 到 handle 函数内部 `deferred.onFulfilled(value)` 其实执行的是 `resolve2({...})` resolve2中的 `deferred` 保存的是的 `uerInfo => {}` 匿名函数和 `resolve3`; `deferred.resolve` 其实执行的是 `resolve5`, 由于 resolve3 和 resolve5 中的 `deferred` 都是空的于是完成整个流程.

接下来再加入错误处理和异常判断:
```javascript
function Promise(fn) {
    var state = 'pending',
        value = null,
        deferreds = [];

    this.then = function (onFulfilled, onRejected) {
        return new Promise(function (resolve, reject) {
            handle({
                onFulfilled: onFulfilled || null,
                onRejected: onRejected || null,
                resolve: resolve,
                reject: reject
            });
        });
    };

    function handle(deferred) {
        if (state === 'pending') {
            deferreds.push(deferred);
            return;
        }

        var cb = state === 'fulfilled' 
            ? deferred.onFulfilled 
            : deferred.onRejected,
                ret;
        if (cb === null) {
            cb = state === 'fulfilled' 
                ? deferred.resolve 
                : deferred.reject;
            cb(value);
            return;
        }
        try {
            ret = cb(value);
            deferred.resolve(ret);
        } catch (e) {
            deferred.reject(e);
        }
    }

    function resolve(newValue) {
        if (newValue && (typeof newValue === 'object' || typeof newValue === 'function')) {
            var then = newValue.then;
            if (typeof then === 'function') {
                then.call(newValue, resolve, reject);
                return;
            }
        }
        state = 'fulfilled';
        value = newValue;
        finale();
    }

    function reject(reason) {
        state = 'rejected';
        value = reason;
        finale();
    }

    function finale() {
        setTimeout(function () {
            deferreds.forEach(function (deferred) {
                handle(deferred);
            });
        }, 0);
    }

    fn(resolve, reject);
}
```
**之前的文字描述不好理解, 所以还是画了一个执行流程图**
![promise执行过程](/img/Promise执行过程.png)

没有画 `reject` 的情况, 因为`reject` 跟 `resolve` 的流程是一样的, 两个一起画显得更乱就单独把 `resolve` 拎出来. 在加入 `handle` 和 `resolve` 在 promise 函数中作为内部方法后实在不易理解. 主要就是通过闭包来保存 promise 对象的变量引用, 将回调函数和 resolve 函数保存在缓存队列中, 在通过 resolve 完成链式调用.
    
## 其他实现
看完之前的那篇文章实在觉得有点难理解, 于是又找了其他的实现方式[JS Promise的实现原理](http://bruce-xu.github.io/blogs/js/promise)比之前那篇更好理解一些, 所以还是记录一下好了.

> 这篇文章作者说到的 `Promise` 重点:
> 1. `Promise` 是一个承诺, 所以不管成功与否都要有一个执行结果, 因此 Promise 构造函数有一个函数类型的参数 `resolver` 来作为与该 promise 对象关联的任务.
> 2. 三种状态不可逆转.
> 3. `resolver` 函数封装了需要执行的异步操作, 内部: `resolve` 和 `reject` 两个参数; 分别代表执行成功和执行失败需要执行的操作.
> 4. `then` 提供成功或失败的响应处理, 于是有了`onResolve` 和 `onReject`.
> 5. `then` 方法返回一个新的 promise(bridge promise), 提供链式操作及串行, 如果直接返回 this 那么就是并行显然不符合我们的需求. 前一个 promise 需要知道下一个 promise 对象是谁及其任务引用, 而后一个 promise 要提供一个给前一个 promise 成功或失败时需要执行的任务, 因此添加一个闭包
> 6. `makeCallback` 调用将 promise 及其关联任务传递进去, 返回一个新函数, 前一个 promise 对象就将持有返回函数的引用, 调用返回函数时就能访问到 promise 对象和关联任务.
> 7. `resolve` 和 `reject` 函数在异步成功或失败的时候调用, 并传递成功的数据和失败的原因.
> 8. `run` 函数执行异步相关的回调函数.

### 代码结构

- 构造函数

```javascript
function Promise (resolver) {
    // 状态
    this._status = 'pending'
    
    // 成功队列
    this._doneCallbacks = []
    // 失败队列
    this._failCallbacks = []
    
    // 传递关联任务
    resolver(resolve, reject)
}
```
- `then` 方法

```javascript
this.prototype.then = function (onResolve, onReject) {
    // 添加闭包调用
    let promise = new Promise(_ => {})
    
    // 保存对上一个 promise 的引用
    this._doneCallbacks.push(makeCallback(promise, onResolve, 'reslove'))
    this._failCallbacks.push(makeCallback(promise, onReject, 'reject'))
    
    return promise
}
```
- `makeCallback` 函数

```javascript
// promise 对象/ 回调函数(关联任务)/ 类型
function makeCallback (promise, callback, action) {
    return function promiseCallback (value) {
    
    }
}
```
- `resolve` 函数和 `reject` 函数
这两个函数都需要一个参数来接收结果, 由于状态只能转换一次所以两个函数都需要判断状态.

```javascript
// 成功
// promise: 属于哪个 promise 对象
// data: 异步操作的结果
function resolve (promise, data) {
    // 已经被 resolve 过的话直接返回
    if (promise._status !== 'pending') return
    
    // 修改 promise 状态
    promise._status = 'fulfilled'
    // 保存异步操作的值
    promise._value = data
    
    // 执行相关回调函数
    run(promise)
}

// 失败
// promise: 属于哪个 promise 对象
// reason: 失败原因
function reject (promise, reason) {
    if (promise._status !== 'pending') return
    
    promise._status = 'rejected'
    promise._value = reason
    
    run(promise)
}
```
- `run` 函数
用来执行异步相关的回调函数.

```javascript
function run (promise) {
    // then 方法中也会调用, 此处再做一次判断
    if (promise._status === 'pending') return
    
    let value = promise._value
    // 是否成功
    let callbacks = promise._status === 'fulfilled'
        ? promise._doneCallbacks
        : promise._failCallbacks
    
    // 这里需要异步
    setTimeout(_ => {
        // 执行回调函数
        callbacks.map(cb => cb(value))
    }, 0)
    
    promise._doneCallbacks = []
    promise._failCallbacks = []
}
```
**`run` 函数中的 callbacks 就是 `makeCallback` 所返回的函数**

- 完善 `makeCallback` 函数

```javascript
function makeCallback (promise, callback, action) {
    return function promiseCallback (value) {
        // 如果 callback 是个函数, 使用前一个 promise
        // 传递的值作为 callback 的参数
        if (typeof callback === 'function') {
            let x
            try {
                x = callback(value)
            }catch (e) {
                // 异常时, 用当前 promise 的 reject
                reject(promise, e)
            }
            
            // 如果 callback 返回的是当前的 promise
            // 要抛出异常
            if (x === promise) {
                let reason = new Error('Error: return value could not be same with the promise')
                reject(promise, reason)
            }
            // 如果返回值是一个 promise 对象
            // 则当返回 promise 对象被 reoslve/ reject后
            // 再执行当前的 promise 的 resolve/ reject
            else if (x instanceof Promise) {
                x.then(
                    function (data) {
                        resolve(promise, data)
                    },
                    function (reason) {
                        reject(promise, reason)
                    }
                )
            }else {
                let then
                (function resolveThenable (x) {
                    // 如果返回的是一个 Thenable 对象
                    if (x && (typeof x === 'object' || typeof x === 'function')) {
                        try {
                            then = x.then
                        }catch (e) {
                            reject(promise, e)
                            return
                        }
                        
                        if (typeof then === 'function') {
                            // 调用 Thenable 对象的 then方法
                            // 传递进去的 resolvePromise 和 rejectPromise 以及下面两个匿名函数
                            // 可能会重复调用, 但是规范只能有其中一个被调用一次, 其他要被忽略
                            let invoked = false
                            try {
                                then.call(x,
                                    function (y) {
                                        if (invoked) return
                                        invoked = true
                                        
                                        // 避免两个 promise 恒等
                                        if (y === x) {
                                            throw new Error('Error: return value could not be same with the promise')
                                        }
                                        
                                    // y 有可能还是 thenable 对象    
                                    resolveThenable(y)
                                    },
                                    function (e) {
                                        if (invoked) return
                                        invoked = true
                                        
                                        reject(promise, e)
                                    }
                                )
                            }catch (e) {
                                    // 如果`resolvePromise`和`rejectPromise`方法被调用后，再抛出异常，则忽略异常
                                    // 否则用异常对象reject此Promise对象
                                if (!invoked) {
                                    reject(promise, e)
                                }
                            }
                        }else {
                            resolve(promise, x)
                        }
                    }else {
                        resolve(promise, x)
                    }
                }(x))
            }
        }
        // 如果没有传 callback直接使用前一个 promise
        // 传过来的值 resolve/reject 当前 promise 对象
        else {
            action === 'resolve'
                ? resolve(promise, value)
                : reject(promise, value)
        }
    }
}
```

#### 总结
我一开始是先看的**剖析 Promise 之基础篇**, 前半部分确实很亲民, 但是到加入 `hanlde` 和改造 `resolve` 函数 就开始懵了, 然后懵懵懂懂的去看**JS Promise 的原理实现**, 不过在最复杂的 `makeCallback` 函数中解释有点一笔带过的意思. 不过看了这篇帮助我理解之前的**剖析 Promise 之基础篇**, 就返回去看基础篇画了一遍流程图才算看懂, 里面的闭包用的太精妙了.

> 参考文章
> [剖析 Promise 之基础篇](https://tech.meituan.com/promise-insight.html)
> [JS Promise 的原理实现](http://bruce-xu.github.io/blogs/js/promise)
> [剖析Promise内部结构](https://github.com/xieranmaya/blog/issues/3)

Created on 2017-12-26 by Cara
