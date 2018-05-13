---
title: JavaScript 关于运算的小技巧
date: 2017-10-17 22:10:33
tags:
- JavaScript
categories:
- 笔记📒
- 复习
---
> #### 1. 使用 `!!` 操作符转换布尔值
用于检查一个变量是否存在或者是有效值, 对变量使用 `!!variable` 来验证,
只要变量的值为: 0, null, undefined, NaN都将返回 false, 反之返回 true. 如: 

```javascript
 class Foo {
    constructor ( count ) {
        this.cash = count;
        this.myCash = !!count;
    }
}
var emptyFoo = new Foo(0);
console.log(foo.cash); // 0
console.log(foo.myCash); // false

var foo = new Foo(100);
console.log(foo.cash); // 100
console.log(foo.myCash); // true
```

> #### 2. 使用 `+` 将字符串转换为数字
只适合将字符串数据转换为数字, 给后台传数据的时候经常用到, 
如果不是字符串数据会返回 NaN.

```javascript
var toNumber = strNumber => {
    return +strNumber;
}
console.log( toNumber("123") ); // 123
console.log( toNumber(" abc")); // NaN

// Date也可以使用
console.log( +new Date() ); // 返回时间戳
```

> #### 3. 并符条件
经常用到这样的条件判断.

```javascript
if (isConcat) {
    Login()
}
```
可以简写成这样 ` isConact && Login() `

### 4. 获取数组中的最后一个元素
`Array.prototype.slice (begin, end)` 经常用这样的方式来截取数组的元素, 
如果不设置 end 的值, 那么默认会将数组的长度作为 end 值. 
如果将负数作为参数的 begin 值, 就可以获取数组的最后一个元素.

```javascript
var arr = [1,2,3,4,5];
console.log( arr.slice(-1) ); // [5]
conosle.log( arr.slice(-2) ); // [4,5]
以此类推
```
> #### 5. 截断数组
用来锁定数组的长度, 删除数组中的一些元素. 比如数组一共有10个元素, 
但我只需要前5个元素, 就可以通过 `array.length = 5 ` 来截断数组, 如: 

```javascript
var arr = [1,2,3,4,5];
console.log( arr.length); // 5
arr.length = 3;
console.log( arr.length ); // 3
console.log( arr ); // [1,2,3]
```

> #### 6. 将NodeList转换为数组
如果通过 ` doucment.querySelectorAll('p') ` 获取元素, 它返回的是一个DOM元素的
数组 ( NodeList ) 对象, 但是这个数组不具有数组的功能,
比如 ` push() / sort() ` 等. 这就需要将这个 NodeList 转换为真正的数组.
可以使用 ` [].slice.call( NodeList ) ` 来实现. 如:

```javascript
var els = document.querySelectorAll( 'p' );
[].slice.call( els );
OR:
var arrElement = Array.from( els );
ES6:
var arr = [...els]
```
> #### 7. 数组元素重排
```javascript
var list = [1,2,3];
console.log(list.sort(()=>{ Math.random() - 0.5}))
```

Created on 2017-6-14 by Cara
