---
date: 2016-02-11T13:14:14+08:00
description: ""
disqus_identifier: 20160002
slug: beego-webframework-analysis-plan
tags:
- Golang
- beego
title: beego框架源码解读计划
topics:
- 编程语言与开发
book: beego源码解读
---

大家新年好，今天大年初四，规划下对于开源 web 框架 beego 源代码解读计划安排。在年前便一直在想如何进行一次明确的源码解析。但一直拖至今日才安排。也在思考如何进行学习是合适的。
<!--more-->

### 目标
有两个目的，一个是针对自己，一个是针对 beego 使用者。
+ 通过分析 beego 源代码来进一步了解该框架，同时也能从中发现不足，来修复并 Pull Request。一步一步参与 beego 的维护工作。
+ 通过分享 beego 相关内容，来推广 beego 发展。让更多人了解使用 beego 开源框架。

### 工作计划

beego 是基于八大独立的模块构建的，是一个高度解耦的框架。当初设计 beego 的时候就是考虑功能模块化，用户即使不使用 beego 的 HTTP 逻辑，也依旧可以使用这些独立模块，例如：你可以使用 cache 模块来做你的缓存逻辑；使用日志模块来记录你的操作信息；使用 config 模块来解析你各种格式的文件。所以 beego 不仅可以用于 HTTP 类的应用开发，在你的 socket 游戏开发中也是很有用的模块，这也是 beego 为什么受欢迎的一个原因。大家如果玩过乐高的话，应该知道很多高级的东西都是一块一块的积木搭建出来的，而设计 beego 的时候，这些模块就是积木，高级机器人就是 beego。至于这些模块的功能以及如何使用则按照基础模块依次展开分析。

### beego 八大模块

1. cache
2. config
3. context
4. httplibs
5. logs
6. orm
7. session
8. toolbox

### beego MVC
beego 是典型的 MVC 架构，各模块相互配合完成各项工作，故我也针对此，从下至上顺序来对 beego 源码解读。
![beego mvc 执行逻辑](http://beego.me/docs/images/flow.png)

### 内容安排

+ Config设计
+ 缓存处理
+ 日志管理
+ ORM设计
+ HTTP工具
+ 会话管理
+ ADMIN工具
+ MVC架构
+ 页面模块
+ 路由表管理

