---
title: Flex 弹性盒子
date: 2017-11-23 19:15:28
tags: 
- CSS
categories: 
- 复习
---
Flex 是 Flexible box 的缩写(弹性布局), 用来为盒模型提供最大灵活性的布局, 更为方便的实现响应式布局.

> 如何使用 Flex  
任何一个容器都可以指定为一个 Flex 布局.  

```css
.box {
	display: flex;
}
/* 行内元素 */
.box{
	display: inline-flex;
}
/* 内核为 webkit 的浏览器 */
.box{
	/* Safari */
	display: -webkit-flex;
	display: flex;
}
```

设定 Flex布局之后的子元素`float`/ `clear`/ `vertical-align`都将失效.

#### 基本概念
所有采用 Flex 布局的的元素都称为 Flex 容器, 它的所有资源都为 Flex 容器的容器成员, 成为 Flex 项目.
Flex 容器都存在两根轴: 水平的主轴(main axis)和垂直的交叉轴(cross axis), 项目默认沿主轴排列. 每个项目占主轴的空间称为`main size`, 占交叉轴的空间称为`cross size`.

#### Flex 容器的属性

- flex-direction
项目排列在主轴的方向 

    ```css
    .box {
    	flex-direction: row | row-reverse | column | column-reverse;
    }
    ```
	- row: 默认值, 主轴为水平方向, 起点为左边
	- row-reverse: 主轴为水平方向, 起点为右边
	- column: 主轴为交叉轴方向, 起点在顶部
	- column-reverse: 主轴为交叉轴方向, 起点在底部
	
- flex-wrap
项目在主轴上的换行方式

    ```css
    .box {
    	flex-wrap: nowrap | wrap | wrap-reverse;
    }
    ```

	- nowrap: 默认值, 不换行
	- wrap: 换行, 第一行在上面
	- wrap-reverse: 换行, 第一行在下面
	
- flex-flow
该属性为`flex-direction`和`flex-wrap`的缩写, 默认值为`row nowrap`

```css
.box {
	flex-flow: flex-direction || flex-wrap;
}
```

- justify-content
项目在主轴上的对齐方式, 所以它的对齐方式跟轴的方向有关

    ```css
    .box {
    	justify-content: flex-start | center | space-around | space-between | flex-end;
    }
    ```

	- flex-start: 默认值, 左对齐
	- center: 居中
	- space-around: 左右居中对齐, 项目之间间隔为项目与边框的间隔大一倍
	- space-between: 两端对齐
	- flex-end: 右对齐 

- align-items
项目在交叉轴在的对齐方式, 与交叉轴方向有关

    ```css
    .box {
    	align-items: flex-start | center | baseline | stretch | flex-end;
    }
    ```

	- flex-start: 交叉轴起点对齐
	- center: 居中
	- baseline: 项目第一行文字基线对齐, 类似在文字的下划线位置
	- stretch: 默认值, 项目没有高度或为`auto`时, 占满整个容器
	- flex-end: 终点对齐
    
- align-content
多根轴线对齐方式. 如果项目只有一根轴线, 该属性则不起作用. 跟`align-items`容易混淆, 区别在于 `align-content`项目有多行时起作用, 而`align-items`项目单行就可起作用.

    ```css
    .box {
    	align-content: flex-start | center | space-around | space-between | stretch | flex-end;
    }
    ```

	- flex-start: 交叉轴对齐方式
	- center: 居中
	- space-around: 左右居中对齐
	- space-between: 两端对齐
	- stretch: 默认值, 项目轴线占满整个交叉轴

#### Flex 项目属性

- order
定义项目的排列顺序, 数值越小越靠前, 默认为0

    ```css
    .item {
    	order: <number>;
    }
    ```

- flex-grow
定义项目的放大比例, 默认为0(有剩余空间也不放大). 如果所有项目都为1, 它们将等分剩余空间; 如果其中一个项目为2, 其它项目为1, 那么属性为2的项目占据的剩余空间将比属性为1的项目大一倍

    ```css
    .item {
    	flex-grow: <number>;
    }
    ```

- flex-shrink
定义项目的缩小比例, 默认为1(如果空间不足, 将缩小). 如果所有项目都为1, 当空间不足时, 所有项目都将等比例缩小; 如果其中一个项目为0, 其它项目为1, 那么当空间不足时, 前者将不缩小

    ```css
    .item {
    	flex-shrink: <number>;
    }
    ```

- flex-basis
定义属性在分配多余空间之前项目占据的主轴空间(main size). 浏览器会根据这个属性计算是否有多于空间. 默认值为 auto(项目本身的大小). 如果不使用`box-sizing`来改变盒模型的话, 该属性将决定项目的内容盒(content-box)的宽或者高(宽或高取决于主轴的方向).

    ```css
    .item {
    	/* size 的单位可以为 px/ em/rem 百分比都行(负值无效) */
    	flex-basis: <size> | <content>;
    }
    ```

	- size: 跟宽高设置一样, 不多说.
	- content: css3 的几个 width 新属性. fill-available/ max-content/ min-content/ fit-content, 兼容性 IE 不支持.
- flex
该属性为`flex-grow`/`flex-shrink`和`flex-basis`的缩写, 默认值为 `0 1 auto`, 后两个属性可选

    ```css
    .item {
    	flex: none | flex-grow flex-shrink flex-basis
    }
    ```
- align-self
允许单个项目与其他项目有不同的对齐方式, 可覆盖`align-items`属性, 默认值为 auto, 继承父级元素的`align-items`属性, 没有的话等同于`stretch`

    ```
    .item {
    	align-self: auto | flex-start | center | stretch | baseline | flex-end;
    }
    ```
除了 `auto`以外其他属性值跟`align-items`一样就不再重复了.

> 说一下 CSS3 width 的几个新属性  
> 需要带有私有前缀  

- fill-available
让元素自动100%得填充父级宽度, 让元素表现得像块级元素一样, `inline-block`的元素也可以自动填充, 顺便再父级加一个`line-height`让元素垂直居中.
- max-content
它的表现就像设置了`white-space:nowrap`一样, 元素的宽度取子元素中较大宽度的宽度
- min-content
它表示的不是子元素中宽度小的那个宽度, 而是内部元素最小宽度值最大的那个元素作为最终宽度. 比如: 一个`div`中, 有一张`img`的宽度为200px, 有一个`p`里面的文字可多可少里面有英文(如果文本全是中文, 最小宽度值就是一个中文的宽度, 如果夹杂英文, 由于英文默认不换行, 所以最小宽度值是最长单词的宽度), 所以假设这里英文的最长单词不足200px, 那么最终的元素宽度就为200px
- fit-content
设置该属性后, 不需要固定宽度, 在父级元素`margin: 0 auto`就可以实现水平居中了

Created on 2017-11-23 By Cara