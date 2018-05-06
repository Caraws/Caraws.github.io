---
title: cookie localStorage and sessionStorage
date: 2017-06-11 18:04:09
tags: 
- JavaScript
categories:
- 复习
---
浏览器的数据缓存, 常用的三种方式cookie/localStorage/sessionStorage

### 1. cookie

> 全称叫做 HTTP Cookie, 通常我们直接叫做Cookie, 最初是在客户端用于存储会话信息的. 该标准要求服务器对任意 HTTP 请求发送 Set-Cookie Http 头作为响应的一部分, 其中会包含会话信息.这种 HTTP 的响应以 key-value组成的一个cookie, 且在传送中都必须是URL编码  ------**JavaScript高级程序设计**                             

- Cookie 的数据大小限制不能超过4k
- 每次 HTTP 请求都会携带 Cookie,所以Cookie只适合保存很小的数据
- 默认 Cookie 在浏览器会话结束时将所有 Cookie删除, 也可以自己设置删除时间

#### JavaScript 中的Cookie
    
* document.cookie 可以设置一个新的 cookie 字符串, key 和 value是必须的

> `document.cookie = 'name=Cara';`
    
由于每次客户端向服务器发送请求的时候都会发送这个 cookie , 所以最好每次设置的时候都用`encodeURI()`进行编码, `decodeURI()`来解码

### 2. localStorage
在 HTML5 规范中作为持久保存客户端数据的方案取代了globalStorage(以域作为访问规则), 要访问同一个 localStorage 对象, 页面必须来自同一个域名(子域名无效), 使用同一种协议, 在同一个端口上. 就相当于 globalStorage[location.host].


- 使用方法储存数据

    > localStorage.setItem('name', 'Cara')
    
- 使用属性储存数据

    > localStorage.book = 'JavaScript'
    
- 使用方法读取数据

    > localStorage.getItem('name')
    
- 使用属性读取数据

    > var book = localStorage.book

- 删除储存数据

    > localStorage.removeItem('name')
    
> localStorage 和 sessionStorage 虽然也有数据储存大小限制, 但比 cookie 要大很多, 可以达到5M或更大. localStorage的数据始终有效, 即使浏览器关闭也会一直保存, 所以只要不 clear() 或者 removeItem() 数据就可以永久保存

### 3. sessionStorage
sessionStorage对象储存特定于某个会话的数据, 也就是说数据只保存到浏览器关闭.

* sessionStorage的用法跟localStorage的用法一样

### Storage类型
Storage 类型提供最大的储存空间 ( 因浏览器而异 ) 来储存名值对儿, 有如下方法:

- clear(): 删除所有值; ( Firefox 未实现 )

- getItem(name): 根据指定的名字 name 获取对应的 value

- key(idnex): 获取 index 位置的值对应的名字

- removeItem(name): 删除由name指定的名值对儿

- setItem(name, value): 设置指定的name及对应的value

>  可以只用 length 来判断有多少名值对儿存放在 Storage 对象中






