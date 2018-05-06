---
title: 关于移植 HEXO
date: 2017-10-14 18:47:02
tags: hexo
categories:
- 笔记📒
---

迫于新买了台电脑, 所以要移植一下台式上的博客. 由于老是忘记 hexo 的官网所以在这儿附一个[hexo 官网](https://hexo.io/docs/)

### 准备工作
将原电脑上的博客源文件上传至 GitHub, 只要源文件相同那么博客会认为是同一个项目, 包括如下文件:

<img src='http://ordnc4i2b.bkt.clouddn.com/blog_source.jpg' alt='blog_source_files'>

其实主要需要的文件是`_conifg.yml`/ `source`文件夹和 `themes`文件夹, 
接下来新建一个仓库, 把这些文件上传上去.

### 移植

1. 在新电脑上克隆刚刚创建的这个项目

<img src='http://ordnc4i2b.bkt.clouddn.com/17_10_14_blog.png' alt='git_clone_files'> 

2. 初始化新的博客
    1. 新建一个文件夹
    2. 命令行进入这个文件夹 `cnpm install hexo-cli -g` 全局安装 hexo(默认已经装过 [淘宝镜像](https://npm.taobao.org/))
    3. 然后`cnpm install` 安装一下依赖项
    3. 初始化完成之后, 把我们克隆下来的文件拷贝到这个文件夹, 覆盖掉初始化的文件.
    4. 如果不想改之前的主题直接`hexo s` 就可以在本地跑起来了.

### 更换主题
其实更换 hexo 主题很简单, 首先我们先去官网选择一个自己喜欢的主题. [主题官网](https://hexo.io/themes/)

1. 主题

这里我选了个很简洁的主题[Anatole](https://github.com/Ben02/hexo-theme-Anatole), 进入这个主题的仓库, 一般作者都会告诉怎么样安装和一些必要的插件.

2. 安装主题
这里我直接下载 .zip 压缩包, 解压后把文件名更改为 anatole, 然后移动到你的博客路径下的`/themes` 文件夹内, 然后修改博客根目录下`_config.yml`中的`theme:`属性改为 anatole. (如果你是其他主题就改为你拷贝进来的主题文件夹的名称就好)

3. 安装插件
`npm install --save hexo-renderer-jade hexo-generator-archive`

4. 预览及部署命令

```javascript
// 新建文章
hexo new 'xxx'

// 本地预览
hexo s (hexo server)

// 生成静态文件
hexo g (hexo generate)

// 部署网站 (部署网站前需要先生成静态文件)
hexo d (hexo deploy)

// 清除缓存
hexo clean
```

如果部署时报错 Not Found git, 有可能时两个原因: 
1. 博客根目录下 `_config.yml` 中 `deploy.type` 如果是 `github` 那就改为 `git` (3.0版本更新的问题)
2. 没有安装插件, 执行命令`npm install hexo-deployer-git --save`
    




