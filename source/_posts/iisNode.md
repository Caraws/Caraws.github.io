---
title: 关于将Node.js部署在IIS服务器
date: 2017-05-28 20:57:06
tags: 
- JavaScript 
- Node.js
categories:
- 笔记📒
---

### 起因
在工作临时遇到后台管理系统需要用到Node.js做中转服务, 达到即时通讯的目的.

由于刚刚转行基础实在太差, 在此做下记录.

### 依赖项

* Node.js

* [URL rewrite](http://www.iis.net/download/URLRewrite)

* [IISNode](http://go.microsoft.com/?linkid=9784334)

### 资源

[GitHub资料](https://github.com/tjanczuk/iisnode)

### 使用

* #### 安装`URL rewrite` 时, 可能会出现安装失败, 需要改下注册表.
	
```
	打开注册表编辑器(win + r `regedit`)，在HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\InetStp位置

把MajorVersion的值改为9之后，就可以安装了，安装完成之后，再把MajorVersion的值改回10，重启一下iis

```

* #### 安装IISNode
	一路`下一步`就行

* #### 配置`web.config`文件
```
<configuration>
	<system.webServer>
		
		<!--indicates that the app.js file is a node.js application
			to be handled by the iisnode module -->
		<handlers>
			<add name="iisnode" path="app.js" verb="*" modules="iisnode" />
		</handlers>

		<rewrite>
			<rules>
				<rule name="SendToNode" patternSyntax="ECMAScript">
					<match url="/*" />
					<action type="Rewrite" url="app.js" />
				</rule>
			</rules>
		</rewrite>
		<webSocket enabled="false"/>
	</system.webServer>
</configuration>
```

* #### 部署
	在iis部署之后会自动生成一个iisnode文件夹, 里面存储debug信息,

因为部署在iis之后, 同时启动Node会报错或者不会打印任何东西.




