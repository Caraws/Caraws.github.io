---
title: Inheritance 继承
date: 2017-10-16 22:37:47
tags:
- JavaScript
categories:
- 复习
- 笔记📒
---
> # 继承
在那些基于类的语言, 继承是一种代码重用的形式, 如果一个新的类与一个已存
在的类拥有大部分相同的功能, 那么就只需要说明两者之间的区别即可. 但是
 JavaScript 并没有类的概念, 但是有很多代码重用的模式, 它可以模拟类的
模式, 也可以支持其他的模式.

### 伪类
在 C++ 和 Java中都是用 new 命令来生成示实例, 在使用 new 命令时都会调用类的
构造函数(constructor), 因此 Brendan Eich ( JavaScript 作者)将 new 引入了 JavaScript,
从原型对象上生成一个实例对象. 因为 js 没有类, 所以在 new 命令后面跟的是构造函数,
而不是像 Java 一样跟的是类. 简单的来说就是通过构造器生产对象.

```javascript
// 先扩展一个方法, 懒得每次打prototype
// 这个不是必须的
Function.prototype.method = function (name, fn) {
	this.prototype[name] = fn;
	return this	
}

// 语言精粹里的例子 (其实这一步就是在模仿 new 的实现)
Function.method('new', function () {

	// 创建一个新对象, 它继承构造器函数的原型对象
	let that = Object.create(this.prototype);

	// 调用构造器函数, 将 this 绑定到新对象上
	let other = this.apply(that, arguments);

	return (typeof other === 'object' && other) || that
})

// 现在定义一个构造器
let Bar = function (name) {
	this.name = name;
}

// 扩展这个构造函数的原型
Bar.prototype.get_name = function () {
	return this.name
}
Bar.prototype.say = function () {
	this.saying || '';
} 

// 然后构造一个实例
let myBar = new Bar('妲己');
console.log(myBar.get_name()) // 妲己

/**
 *  掩盖掉丑陋的 prototype
 */

// 借助一个辅助函数
Function.method('inherit', function (parent) {
	this.prototype = new parent();
	return this
})

// 重新定义一个构造函数
// 去继承上面的 Bar
let Foo = function (name) {
	this.saying = 'wow';
}
// 这里 inherit 和 method 都直接返回 this
// 所以可以采用联级
.inherit(Bar)
.method('noise', function () {
	return this.name + this.say()
})

```
伪类模式在通过new 命令生产对象时, 会产生内存浪费. 如上面的例子, Foo 构造函数
就回去重复构造器 Bar 已经完成的工作.

### 原型
在一个纯粹的原型模式中, 将摒弃类专注于对象. 基于原型的继承相比于类的
继承在概念上更为简单: 一个新对象可以继承一个旧对象的属性. 通过构造一个有用
的对象开始, 接着可以构造出更多和这个对象相似的对象.   ----<JavaScript语言精粹> 

```javascript
// 先构造一个基础对象
const baseObject = {
	name: 'base',
	get_name: function () {
		return this.name
	}
}

// 接下来构造定制化的对象
let myObj = Object.create(baseObject);
myObj.name = 'wow';
myObj.saying = 'Hi';
myObj.purr = n => {
	let str = '';
	for (let i = 0; i < n; i++) {
		if (str) {
			str += 'hello'
		}
		str += 'world'
	}
	return str
}
```

这是一种'差异化继承', 通过制定一个新的对象, 指明它与基本对象的不同.



### 函数化
以上两种继承模式都没有实现私有化, 也就是说所有的变量和方法都是公开的, 所以就
可以开始运用模块模式. 这个函数主要分为四个步骤:

1. 创建一个新的对象

2. 定义私有属性.

3. 给这个新对象扩充特权函数 (暴露接口)

4. 返回这个对象

```javascript
// spec 对象包含构造器所需要的所有信息
// my 对象允许其他构造器分享他们的私有属性
// 以便在我们的构造器中使用
let constructor = (spec, my) => {
	let that = {}; // 私有实例变量

	let my = my || {};
	
	// 扩展共享的变量和方法
	...

	that = 一个新对象

	// 给 that 添加特权方法 
	...

	// 返回这个对象
	return that
}
```

> 语言精粹里的例子

1. 构造器
```javascript
const mammal = spec => {
	let that = {};

	that.get_name = _ => spec.name;
	that.says = _ => spec.saying || '';

	return that
}

let myMammal = mammal({name: 'Herb'});
```

2. 另一个构造器
```javascript
let cat = spec =>{
	spec.saying = spec.saying || 'meow';
	// 继承
	let that = mammal(spec);
	
	that.purr = n => {
		let i, s = '';

		for (i = 0; i < n; i++) {
			if (s) {
				s += '-';	
			}
			s += 'r';
		}
		return s
	};
	that.get_name = _ => {
		return that.says() + ' ' + spec.name + ' ' + that.says();
	}

	return that
}

let myCat = car({name: 'Henrietta'});
```

### 超类
以上函数化的方式还不能够调用父类的方法并向父类方法传递参数. 以下是测试代码
在`语言精粹`的例子的基础上稍作改动, 便于自己理解:

```javascript
// 构造器
const mammal = spec => {
	let that = {};

	that.get_name = _ => spec.name;
	that.says = _ => spec.saying || 'Hi';

	return that
}

let myMammal = mammal({name: 'Cara'});

console.log(myMammal.says()) // 'Hi'


// 构造器2
const cat = spec => {
	spec.saying = spec.saying || 'meow';

	// 此时 that 已经包含: get_name 和 says 方法
	let that = mammal(spec);

	that.purr = n => {
		let str = '';

		for (let i = 0; i < n; i++) {
			if (str) {
				str += '-';
			}
			str += 'r';
		}
		return str
	};
	that.get_name = n => {
		return n || "i dont't have name"
	};

	return that
}

let myCat = cat({name: 'Henrietta'});

// i dont't have name
console.log(myCat.get_name())

// 这个时候 cat 还不能访问父类方法的能力
// 所以超类 super 还是有必要的

// 先扩展两个方法
// 这个方法不能用箭头函数来定义, 否则 this 指向的window
// 这里 this 需要的是被调用的对象
Function.prototype.method =  function (name, fn) {
	this.prototype[name] = fn;
	return this
}

// 定义调用父类的函数
Object.method('superior', function (name) {
	var that = this,
        method = that[name];
    return n => {
    	// this cat function
    	console.log(n)
        return method.call(that, n);
    };
})

// 来试试调用父类
const coolCat = spec => {
	let that = cat(spec);
	let	super_get_name = that.superior('get_name');

	// n 给父类方法传参
	that.get_name = n => {
        return 'like ' + super_get_name.call(this, n) + ' baby';
    };

	return that
}

let myCoolCat = coolCat({name: 'Bix', saying: 'Hi'});

let name = myCoolCat.get_name('this cat function');

console.log(name) // like this cat function baby
```

#### 最后随便提一下
箭头函数的几个使用注意点:

1. 函数体内的 `this` 对象, 就是定义时所在的对象, 而不是使用时所在的对象.

2. 不可以当作构造函数, 也就是说, 不可以使用new命令, 否则会抛出一个错误.
因为箭头函数没有自己的 `this`, 而是继承外层函数的`this`.

3. 不可以使用 `arguments` 对象, 该对象在函数体内不存在.

4. 不可以使用 `yield` 命令, 因此箭头函数不能用作Generator函数.

Ceated on 2017-9-15 by Cara 