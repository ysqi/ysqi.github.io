
---
date: 2016-12-31T11:35:12+08:00
title: "使用Go语言编写的跨平台的emlog整站(首页+文章)静态化工具"
description: ""
disqus_identifier: 1485833712078694729
slug: "shi-yong-Goyu-yan-bian-xie-de-kua-ping-tai-de-emlogzheng-zhan-(shou-xie-wen-zhang-)jing-tai-hua-gon"
source: "https://segmentfault.com/a/1190000000348983"
tags: 
- 静态化 
- emlog 
- golang 
categories:
- 编程语言与开发
---

emlog\_static.go
----------------

### 项目地址： <https://github.com/johnlui/emlog_static.go>

====================

### 功能

1.  新闻博客类非数据交互网站通用首页静态化
2.  emlog博客系统文章页静态化
3.  使用Go语言编写，跨平台

### 条件

1.  需要能够管理服务器，需要编译、运行软件
2.  需要Go语言编译环境，请去[The Go Programming
    Language](http://golang.org/) 下载

### 使用方法

1.  两个文件单独编译，分别运行
2.  emlog\_static\_homepage 首页静态化 使用方法为：

    ./emlog\_static\_homepage -url=<http://example.com/index.php> -t=30

    t为间隔时间，单位是秒

3.  emlog\_static\_articles 文章页静态化 使用方法为：

    ./emlog\_static\_articles -url=<http://example.com/index.php> -c=10
    -n=100

    c为并发数，n为总文章数，即文章id最大值



