---
title: CSS  垂直水平居中大整合
date: 2017-10-14 20:31:30
tags: CSS
categories:
- 复习
---

在平时的布局中常常会用到的垂直水平居中. 下面就来总结几种垂直
水平居中的方法.

> ### 1. 绝对定位水平垂直居中
> 给元素设置绝对定位, 其父级元素为`body`或者指定的相对定位元素. 
> 它的缺点是必须设置宽度值或者高度值; 优点是兼容性挺好, 代码也不
> 多, 不过听说在 windows phone 上不起作用.

```html
<div class='wrapper'>
	<div class='absolute-center'>我是绝对定位<div>
</div>
```

```css
.wrapper{
	width: 100%;
	height: 200px;
	position: relative;
	border: thin solid black;
}
.absolute-center{
	position: absolute;
	top: 0;
	right: 0;
	bottom: 0;
	left: 0;
	margin: auto;
	background: deepskyblue;
	height: 100px;
	line-height: 100px;
	text-align: center;
	color: #fff;
}
```

> 之前一直不懂绝对定位的工作机制到底是怎样, 下面是查资料看到的:

1. 在普通文档流中`margin: auto`相当于`margin-top: 0; margin-bottom: 0;`;
2. `position: absolute`会脱离文档流, 脱离的部分将不会影响文档流中的内容.
3. 给块区域内设置`top: 0; right: 0;bottom: 0;left: 0`, 浏览器将重新分配一个边界框,  这个时候的块区域会占满父级所有可用空间.
4. 为这个块区域设置宽度或高度就可以防止该元素占满整个父级元素, 促使浏览器根据新的边界值重新计算`margin: auto`.
5. 由于内容块被绝对定位, 脱离了正常的内容流, 浏览器会给`margin-top,margin-bottom`相同的值, 使元素块在先前定义的边界内居中.

> ### 2.  可视区域内水平垂直居中
> 把上面例子改为`position: fixed; z-index: 999`, 设置一个较大值的`z-index`; 如果不给这个块级元素设置宽高则会占满整个屏幕.

```html
<div class="fixed-center">固定定位水平垂直居中</div>
```

```css
.fixed-center{
	width: 200px;
	height: 200px;
	position: fixed;;
	z-index: 999;
	background: rgba(0, 0, 0, .3);
	top: 0; 
	left: 0;
	bottom: 0; 
	right: 0; 
	margin: auto;
	line-height: 200px;
	color: #fff;
	text-align: center;
}
```

> ### 3. 边栏垂直居中
> 有时需要将一个内容块固定在屏幕的左侧或者右侧, 可以向内容快加入像这样的 CSS 代码`top: 20px; bottom: auto;`; 由于已经声明了`margin: auto`, 内容块将会垂直居中在你定义的`top`/ `right`/ `bottom`/ `left`边界值内. 可以用`left: 0; right: auto`将内容快固定在左侧; `right: 0; left: auto;`将内容固定在右侧.

```html
<div class="absolute-left">
	绝对定位在左侧
</div>
<div class="absolute-right">
	绝对定位在右侧
</div>
```

```css
.absolute-left {
	position: absolute;
	left: 20px;
	right: auto;
	margin: auto;
	width: 150px;
	height: 80px;
	background: red;
	color: #fff;
	text-align: center;
	line-height: 80px;
}
.absolute-right {
	position: absolute;
	left: auto;
	right: 20px;
	margin: auto;
	width: 150px;
	height: 80px;
	background: green;
	color: #fff;
	text-align: center;
	line-height: 80px;
}
```

> ### 4. 自适应绝对定位居中
> 自适应绝对居中的优势就是对百分比的宽高完美支持, 甚至是`max-width/ min-width`或者是`max-height/ max-width`, 即使加上`padding`也不会影响绝对居中的实现.

```html
<div class="wrapper-responsive">
	<div class="absolute-responsive">
		自适应绝对定位居中
	</div>
</div>
```

```css
.wrapper-responsive{
	position: relative;
	top: 300px;
	width: 200px;
	height: 200px;
	border: thin solid black;
}
.absolute-responsive{
	position: absolute;
	top: 0;
	right: 0;
	bottom: 0;
	left: 0;
	margin: auto;
	width: 60%;
	height: 60%;
	min-width: 50px;
	max-width: 150px;
	padding: 10px;
	text-align: center;
	color: #fff;
	background: skyblue;
}
```

> ### 5. 内容溢出
> 有时内容块的内容过多导致高度大于父级元素, 内容就会显示到块和容器的外面, 这种时候加个给内容块`overflow: auto`即可, 如果内容块没有`padding`的话加个`max-height: 100%`也行.

> ### 6. 可变高度
> 设置`display: table`, 内容块会垂直居上不过水平还是居中. 但是这个在一些浏览器上有问题( 如: IE/ FF); `display: table-cell`配合`vertical-align: middle;`也可实现垂直水平居中.

```html
<div class="wrapper-table">
	<div class="table-center">
		table 水平居中
		内容内容内容内容内容内容内容内容内容内容内
	</div>
</div>
	
<div class="wrapper-table-cell">
	<div class="table-center-cell">
		table-cell 居中
		内容内容内容内容内容内容内容内容内容内容内
	</div>
</div>
```

```css
/* tabel 居中 */
.wrapper-table{
	margin-top: 600px;
	display: table;
	height: auto;
	width: 200px;
	height: 200px;
	border: thin solid black;
}
.table-center{
	width: 80px;
	background: orange;
	color: #fff;
}

/* tabel-cell 居中 */
.wrapper-table-cell{
	margin-top: 600px;
	display: table-cell;
	height: auto;
	width: 200px;
	height: 200px;
	border: thin solid black;
	vertical-align: middle;
}
.table-center-cell{
	width: 100px;
	background: #FFD34E;
	color: #fff;
}
```

> ### 6. 负外边距
> 这个方法应该是很常用的方法了, 已知元素的宽高: 外边距`margin`的值取负数(没有`box-sizing: border-box`的情况下, 包括`padding`), 大小为 `width/ height`的一半, 再加上`top: 50%; left: 50%`, 这个不能自适应哦.
```html
<div class="wrapper-margin">
	<div class="margin-center">
		margin 居中
	</div>
</div>
```

```css
.wrapper-margin{
  position: relative;
	top: 100px;
	border: thin solid black;
	width: 200px;
	height: 200px;
}
.margin-center{
	width: 100px;
	height: 100px;
	background: #105B63;
	position: absolute;
	top: 50%;
	left: 50%;
	margin: -50px auto auto -50px;
	color: #fff;
	text-align: center;
	line-height: 100px;
}
```

> ### 7. 变形
> 这可以说是最简单的方法了, 不仅可以实现绝对居中的想过还可以实现可变宽高度. 内容块定义`transform: translate(-50%, -50%)`带上浏览器前缀, 再加上`top: 50%; left: 50%`即可.

```html
<div class="wrapper-transform">
	<div class="transform-center">
		transform 居中
	</div>
</div>
```

```css
.wrapper-transform{
	position: relative;
	top: 200px;
	border: thin solid black;
	width: 200px;
	height: 200px;
}
.transform-center{
	width: 100px;
	height: 100px;
	background: #BD4932;
	position: absolute;
	top: 50%;
	left: 50%;
	-webkit-transform: translate(-50%, -50%);
	-ms-transform: translate(-50%, -50%);
	-o-transform: translate(-50%, -50%);
	transform: translate(-50%, -50%);
	color: #fff;
	text-align: center;
}
```

> ### 8. Flexbox
> CSS3的新属性也叫弹性盒子, 它不仅可以解决居中的问题还可以分栏或者其他的一些布局问题. 这里不多解释, 下次做一个详细的说明.
```html
<div class="wrapper-flex">
	<div class="flex-center">
		弹性盒子居中
	</div>
</div>
```

```css
.wrapper-flex{
	margin-top: 500px;
	width: 200px;
	height: 200px;
	border: thin solid black;
	display: flex;
	justify-content: center;
	align-items: center;
}
.flex-center{
	background: grey;
	color: #fff;
	text-align: center;
}
```

[预览地址](https://caraws.github.io/Review-JavaScript/CSS/vertical-horizontal-center)

Created on 2017-10-11 by Cara
