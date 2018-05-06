---
title: JavaScript 深浅拷贝
date: 2017-10-16 21:45:01
tags:
- JavaScript
categories:
- 笔记📒
---

> ### 浅拷贝
浅拷贝只能拷贝顶层属性基本数据类型, 也就是如果父对象的属性是一个对象或数组, 那么子对象获取到的只是一个内存地址而不是一个真正的对象, 所以一旦修改父对象也会跟着被篡改.

```javascript
    function shallowCopy ( parent ) {
        let o = {};
        for (let i in parent) {
             o[i] = parent[i]
        }
        return o
    }
```

> ### 深拷贝
深拷贝也就是能够实现数组和对象拷贝, 深拷贝与浅拷贝对比, 深拷贝会在堆区开辟新的一块来储存新的对象. 两个对象对应的是两个不同的内存地址, 所以修改其中一个对象的属性并不会影响到另一个对象的属性. 实现深拷贝也有两种方式: 一种是递归/ 一种是JSON.

1. 递归
```javascript
    function deepCopy (parent, child = {}) {
        for (let i in parent) {
            if (typeof parent[i] === 'object') {
                child[i] = (parent[i].constructor == Array) ? [] : {};
                deepCopy(parent[i], child[i])
            } else {
                 child[i] = parent[i]
            }
        }
    }
```

2. JSON解析(严格的 JSON 格式)
```javascript
    var o = {
        age: '18',
        friends: ['老张', '老王'],
        job: {
           main: '睡觉',
           sub: '躺着'
        }
    };
    var result = JSON.parse( JSON.stringify(o));
    result.name = 'cara';
    result.friends.push('老李');
    console.dir(o);
    console.dir(result)
```

以上两种方式无法解析一下几种情况:
- RegExp 对象
- 函数
- 会摒弃对象的 constructor , 所有构造函数都会指向 Object
- 对象循环引用也会报错

所以要面对不同的对象, 做不同的处理方式, 就需要检查一下对象的类型:

```js
// 检查类型
const Type = _ => {
    const Types = {}
    for (let i = 0, type; type = ['Array', 'Date', 'RegExp'][i++];) {
        Types['is' + type] = (obj) => {
            if (typeof obj !== 'object') return
            return Object.prototype.toString.call(obj) === `[object ${type}]`
        }
    }
    return Types
}
const types = Type()

// 获取正则影响范围
const getRegExp = (reg) => {
    if (reg.globlal) return 'g'
    if (reg.ignoreCase) return 'i'
    if (reg.multiline) return 'm'
}

const clone = (oldObj = {}) => {
    let oldObjs = []

    const _clone = oldObj => {
        let newObj, proto
        // 不同类型, 不同处理方式
        if (types.isArray(oldObj)) {
            newObj = []
        }
        if (types.isDate(oldObj)) {
            newObj = new Date(oldObj.getTime())
        }
        if (types.isRegExp(oldObj)) {
            newObj = new RegExp(oldObj.source, getRegExp(oldObj))
        } else {
            // 斩断原型链
            proto = Object.getPrototypeOf(oldObj)
            newObj = Object.create(proto)
        }
        // 如果之前已经遍历过该对象, 直接返回该对象
        let hasBeen = oldObjs.indexOf(oldObj)
        if (hasBeen !== -1) {
            return oldObjs[hasBeen]
        }
        // 记录操作过的对象
        oldObjs.push(oldObj)
        // 递归
        for (let i in oldObj) {
            newObj[i] = _clone(oldObj[i])
        }
        return newObj
    }
    return _clone(oldObj)
}
```

👇测试一波:
```js
// 测试
function person(name) {
    this.name = name;
}
const Cara = new person('Cara');

function say() {
    console.log('hello world!');
}

const oldObj = {
    a: say,
    c: new RegExp('ab+c', 'i'),
    d: Cara,
};
oldObj.b = oldObj;


const newObj = clone(oldObj)

console.log(newObj.a, oldObj.a) 
// [Function: say] [Function: say]
console.log(newObj.b, oldObj.b)
// { 
//     a: [Function: say], 
//     c: /ab+c/i, 
//     d: person { name: 'Messi' }, 
//     b: [Circular] 
// } 
// { 
//     a: [Function: say], 
//     c: /ab+c/i, 
//     d: person { name: 'Messi' }, 
//     b: [Circular] 
// }
console.log(newObj.c, oldObj.c)
// /ab+c/i /ab+c/i
console.log(newObj.d.constructor, oldObj.d.constructor)
// [Function: person][Function: person] 
```
目前我们上面说的几种坑就得以解决, 不过这还不是最完整的方案, 还有一些 ES6 里面的对象也需要我们做特殊的处理, 不过现在这个版本在日常还是够用了. 另外在生产环境中还是建议用 `lodash` 的 `_.cloneDeep`

Created on 2017-8-15 by cara