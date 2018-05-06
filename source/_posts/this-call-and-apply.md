---
title: this/ call and apply
date: 2017-10-17 23:01:38
tags:
- JavaScript
categories:
- 笔记📒
- JavaScript设计模式
---
在 JavaScript 中, `this `关键字很容易使大家疑惑, 再加上 `Function.prototype.call`和`Function.prototype.apply`这两个方法的广泛运用, 所以很有必要弄清`this`的使用.

>  ### this
> 首先先说一下`this`的概念: `this`总是指向一个对象, 而这个对象具体是谁, 是根据运行时的函数执行的环境动态绑定的, 而非函数被声明时的环境. 当然到现在箭头函数的出现, `this`对不了解的同学们来说, 无疑是添了一把乱… 接下来我们一个一个的来说吧.

#### 1. `this`的指向
我们除开不常用的 with 和 eval 的情况, 具体到实际应用中, `this`的指向大致可以分为以下四种情况.

- 作为对象的方法被调用
- 作为普通函数被调用
- 构造器调用
- 被 call 和 apply 方法调用
下面来用 < JavaScript 设计模式 > 中的例子说明这四种情况.

    1. 作为对象方法调用
    当函数作为对象的方法被调用时, `this`指向该对象:
```javascript
let name = 'window';
let obj = {
    name: 'cara',
    getName: function () {
        console.log(this === obj)
        console.log(this.name)
    }
}
obj.getName() // true 'cara'
```

    2. 作为普通函数调用
    当函数不作为对象的方法被调用时, 也就是我们平常说的普通函数的方式, 此时的`this`总是指向全局对象.
```javascript
window.name = 'window';
let getName = function () {
    return this.name
}
console.log(getName()) // 'window'

// 或者

window.name = 'window';
let obj = {
    name: 'obj',
    getName: function () {
        return this.name
    }
}
// 这里 obj 的 getName方法赋值给了一个变量
// 调用的时候就只会作为一个普通函数调用
let getName = obj.gatName;
console.log(getName()) // 'window'
```
    上面这中作为普通函数调用的方式常常会带来一些困扰, 比如在某个 div 节点的事件函数中, 定义了一个局部的 callback 方法. 而这个 callback 方法我们往往是想让它的 `this`指向 div 节点, 但它内部却指向`window`. 如:
```html
<div id='div1'>我是一个 div</div>
```
    ```javascript
        window.id = 'window';
        document.getElementById('div1').onclick = function () {
            console.log(this.id); // 'div1'
            let callback = function (){
                console.log(this.id); // 'window'	
            }
            callback() // 作为普通函数被调用
        }
    ```
    要解决以上问题其实也很简单, 如下:
```javascript
window.id = 'window';
document.getElementById('div1').onclick = function () {
    console.log(this.id);
    let that = this; // 用一个变量来储存节点的引用
    let callback = function (){
        console.log(that.id); // 'div1'	
    }
    callback() // 作为普通函数被调用
}
```
    3. 构造器调用
    先说说构造器吧, js 中没有类的概念, 但是可以从构造器中创建对象, 同时提供 `new` 运算符, 让构造器看起来更像一个类. Js 中大部分函数都可以当做构造器来使用, 所以它的外表看起来跟普通函数一样, 区别在于被调用的方式. 当用 `new` 运算符调用时, 该函数会返回一个对象. 通常情况下, 构造器里的 `this` 就指向返回的这个对象, 如下:
```javascript
const Myobj = function () {
    this.name = 'cara';
}

let obj = new Myobj();
console.log(obj.name); // 'cara'
```
    但是使用 `new`调用构造器时, 要注意一个问题. 如果构造器显式地返回了一个对象, 那么最终就会返回这个对象, 而不是我们的期望的 `this`:
```javascript
const Myobj = function () {
    this.name = 'cara';
    return {
        name: 'somebody'
    }
}

let obj = new Myobj();
console.log(obj.name); // 'somebody'
```
    如果构造器不显式地返回任何数据或是返回一个非对象类型的数据, 就不会出现上述情况

    4. call 或 apply 方法调用
    跟普通的函数调用相比, 用 call 或者 apply 方法调用可以动态地改变传入函数的 `this`:
```javascript
let obj1 = {
    name: 'cara',
    getName: function () {
        return this.name
    }
};

let obj2 = {
    name: 'ben'
};

console.log(obj1.name); // 'cara'
console.log(obj1.name.call(obj2)); // 'ben'
```
    call 和 apply 方法能够很好的体现 js 的函数式语言特性. 在 js 中几乎每一次编写函数式语言风格的代码都离不开 call 和 apply.

#### 2. 丢失的 this
这是一个经常遇到的问题, 在刚刚开始学习 js 时,`this`的指向常常令我疑惑, 尤其是看到网上关于`this`指向的题目, 简直云里雾里. 下面就来看一些例子吧!

```javascript
const obj = {
    name: 'apple',
    getName: function () {
        return this.name;
    }
}

console.log(obj.getName()); // 'apple'
let getMyName = obj.getName;
console.log(getMyName()); // undefined
```
上面这个例子好理解, 就是通过一个变量来引用`obj.getName`方法, 并且调用 getMyName 时, 就是用的普通函数调用方式, `this`是指向全局 window 的.

    接下来再来看一个稍微复杂一点的例子吧:
```javascript
let name = 'window';
const person = {
    name: 'person',
    showName1: function () {
        console.log(this.name)
    },
    showName2: _ => console.log(this.name),
    showName3: function () {
        return function () {
            console.log(this.name)
        }
    },
    showName4: function () {
        return _ => console.log(this.name)
    }
};
const person2 = {name: 'person2'};

person.showName1();
person.showName1().call(person2);

person.showName2();
person.showName2().call(person2);

person.showName3()();
person.showName3().call(person2);
person.showName3.call(person2)();

person.showName4()();
person.showName4().call(person2);
person.showName4.call(person2)();
```
这个例子可能大家已经很眼熟了, 不过当时我第一次做的时候几乎错了一大半😒…. 现在再拿出来看看其实还是很经典的: 在 person 和 person2 之间疯狂玩 showName 方法. 在给出答案之前我们先看看箭头函数的一些特点:

- 箭头函数不可用作构造函数.
- 不可以使用 `arguments`对象, 如果要用可以使用 rest 参数代替.
- 不能使用`yield`命令, 所以箭头函数也不能作为 Generator 函数.
- 箭头函数的`this`是定义时所在的对象, 而不是执行时所在的对象.

    在最后一点的`this`指向上, 我个人觉得有点误导(也可能是我没理解到位)… 因为如果在对象字面量中的方法是通过箭头函数定义的话, `this`的指向就会和你期望的不一样了. 

```javascript
let name = 'window'
let obj = {
    name: 'obj',
    getName: _ => console.log(this.name)
}
obj.getName(); // 'window'
```
所以我觉得关于理解箭头函数`this`在定义时所在的对象是这样: `this`继承自父级的执行上下文(简单对象即非函数, 是没有执行上下文的), 所以上面例子就是 getName 方法的父级是 obj, 而 obj 的执行上下文是`window`, 因此输出全局对象的 name. 理解了关于箭头函数`this`的指向, 现在再来看看答案吧:

```javascript
person.showName1(); // 'person'
person.showName1().call(person2); //  'person2'

person.showName2(); // 'window'
person.showName2().call(person2); // 'window'

person.showName3()(); // 'window'
person.showName3().call(person2); // 'person2'
person.showName3.call(person2)(); // 'window'

person.showName4()(); // 'person'
person.showName4().call(person2); // 'person'
person.showName4.call(person2)(); // 'perons2'
```

然后来分析一下答案吧:
- 调用 showName1() 
这两个方式好理解, 第一种是通过 person 对象来调用的 showName1 方法, 也就是上面我们说过的**作为对象的方法被调用**, 所以`this`自然指向的是person 对象; 第二种是**被 call 和 apply 方法调用**, 所以`this`指向的是被 call 方法矫正的 person2.

- 调用 showName2()
showName2 方法是一个箭头函数, 根据我们之前说过的箭头函数指向问题来看. 第一种通过 person 对象来调用, 由于 person 是一个简单对象所以这里它的执行上下文就是`window`, 那么就是作为普通函数调用, `this`指向 `window`; 第二种跟第一种是相同的调用方式, 只是把 person 对象换为 person2.

- 调用 showName3()
Person.showName3 是一个高阶函数, 返回了一个匿名函数. 第一种方式相当于直接调用那个匿名函数执行环境就是`window`, 所以`this`指向`window`; 第二种方式通过 person2 来调用 person 的高阶函数, 输出 person2; 第三种先通过 person2 调用 person 的高阶函数, 然后在全局作用域下执行, 因此`this`指向`window`.

- 调用 showName4()
最后这三组调用也是高阶函数, 不过返回的匿名函数用的箭头函数. 前两种方式也就印证了我们之前所说的**箭头函数的 this 继承自父级执行上下文**, 所以前两种都输出 person, 就算第二种方式后面用 call 方法来矫正也是不行的; 第三种也就是通过 person2 来调用执行的 showName4 方法, 自然也就输出 person2 啦.
 
> ### call 和 apply
Function.prototype.call() 和 Function.prototype.apply() 都是非常常用的方法, 在实际开发中和 JavaScript 的设计模式中这两个方法应用广泛. 其实它们的作用是一样的, 只是有传入参数形式不同的区别.

- apply
apply 方法接收两个参数: 第一个参数指定函数体中 `this`的指向; 第二个参数为一个带下标的合集(可以是数组或者类数组), 这个参数将会传递给被调用的函数.
```javascript
let fn = function (a, b, c) {
	console.log(a, b, c); // 1 2 3
}
// 第一个参数为 null 的话, 表示不改变 this 的指向
fn.apply(null, [1, 2, 3]); 
```
- call
call 方法的第一个参数和 apply 方法一样, 指定`this`的指向; 第二个参数不同, call 方法的第二个参数的数量不固定, 从第二参数开始一次按顺序传递给被调用的函数.
```javascript
let fn = function (a, b, c) {
	console.log(a, b, c); // 1 2 3
}
fn.call(null, 1, 2, 3)
```

在我们不关心具体有多少参数被传入函数时, 就可以使用 apply 方法一股脑推过去就行了; 当我们明确的知道有多少参数, 想一目了然的表达形参和实参的对应关系时, 那么就可以用 call 方法.

- 借用其他对象的方法
call 和 apply 经常被用来借用其他对象的方法, 常用的就有借用`Array`的方法来操作`arguments`或者借用构造函数来实现一些类似继承的效果.
	
    1. 借用构造函数
```javascript
let A = function (name) {
	this.name = name
};
// 借用构造函数 A
let B = function () {
	A.apply(this, arguments)
};
B.prototype.getName = function () {
	return this.name
}

let b = new B('cara');
console.log(b.getName); // 'cara'
```
    2. 借用 Array 的方法
```javascript
(function () {
	Array.prototype.push.call(arguments, 3);
	console.log(arguments); // {'0': 1, '1': 2, '2': 3}
})(1, 2)
```
借用的时候要保证两个必要条件: 1. 对象本身要可以存取属性; 2. 对象的 length 属性可读写.
> ### Function.prototype.bind
绑定函数 bind 也是可以用作矫正 `this`的指向, bind 函数会创建一个新的函数(绑定函数), 新函数和目标函数将拥有相同的函数体. 第一个参数绑定`this`的指向, 从第二参数起后面的参数将作为实参绑定到目标函数的形参.

```javascript
let sum = function (a, b) {
	return a + b;
};
let result = sum.bind(null, 1); // 1 将作为实参传入 sum
console.log(result(2)); // 3
```

> 另外当 bind 返回的函数作为构造函数使用的话, 绑定的`this`将被忽略, 实参传入目标函数.

```javascript
let original = function (x) {
	this.a = 1;
	this.b = function () {
		return this.a + x
	}
};

let obj = {
	a: 2
};

let newObj = new (original.bind(obj, 2));
console.log(newObj.a); // 1 => this 的指向被忽略
console.log(newObj.b()); // 3 => 2 被传入 original
```
call 和 apply 方法都是改变`this`指向后立即执行而 bind 可以在你想执行的
时候再执行.

差不多就到这儿吧, 要是有补充再接着写…

Created on 2017-10-17 by Cara

