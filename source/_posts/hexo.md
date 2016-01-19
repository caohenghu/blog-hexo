title: Hexo 简介
date: 2016-01-18 00:00:24
tags:
---

Markdown 是一种轻量级的「标记语言」，它的优点很多，目前也被越来越多的写作爱好者，撰稿者广泛使用。看到这里请不要被「标记」、「语言」所迷惑，Markdown 的语法十分简单。常用的标记符号也不超过十个，这种相对于更为复杂的 HTML 标记语言来说，Markdown 可谓是十分轻量的，学习成本也不需要太多，且一旦熟悉这种语法规则，会有一劳永逸的效果

<!-- excerpt -->

## 1、Hexo安装与设置

**安装之前确保已经安装了node.js**

	npm install hexo -g #-g表示全局安装, npm默认为当前项目安装

**Hexo使用命令:**

    hexo init <folder>  #执行init命令初始化hexo到你指定的目录    
    hexo generate       #自动根据当前目录下文件,生成静态网页
	hexo server         #运行本地服务,

**浏览器输入[http://localhost:4000](http://localhost:4000)就可以看到效果。**

## 2、添加博文

	hexo new "postName"  #新建博文,其中postName是博文题目

**博文会自动生成在博客目录下`source/_posts/postName.md`**

	#文件自动生成格式:
	title: "It Starts with iGaze: Visual Attention Driven Networkingwith Smart Glasses"  #博文题目
	date: 2014-11-21 11:25:38      #生成时间
	tags: Paper                    #标签, 多个标签使用格式[Paper1, Paper2, Paper3,...]
	---

**如果不想博文在首页全部显示, 并能出现 `阅读全文` 按钮效果, 需要在你想在首页显示的部分下添加 `<!--more-->`**

	此处及以上的内容会在首页显示
	<!--more-->
	一下是在首页隐藏的部分