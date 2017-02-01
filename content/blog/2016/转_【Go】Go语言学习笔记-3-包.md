
---
date: 2016-12-31T11:34:19+08:00
title: "【Go】Go语言学习笔记-3-包"
description: ""
disqus_identifier: 1485833659902498406
slug: "【Go】Goyu-yan-xue-xi-bi-ji--3-bao"
source: "https://segmentfault.com/a/1190000003706885"
tags: 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

包是函数和数据的集合，用package关键字定义一个包。

-   文件名不需要与包名一致。

-   包名的约定是使用小写字符。

-   Go包可以由多个文件组成，但是使用相同的package &lt;name&gt;这一行。

-   名称以大写字母起始的时可导出的，可以在包得外部调用。

-   构建包的方法：在\$GOPATH下简历一个目录，然后把这个文件复制到该目录下，然后build和install。

-   共有函数的名字以大写字母开头，私有函数的名字以小写字母开头。

-   文档：在package前的一段注释，会出现在godoc生成的关于包的页面上。

-   单元测试：测试函数以Test开头，运行go test则会调用所有的测试函数。



