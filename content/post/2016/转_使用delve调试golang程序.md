
---
date: 2016-12-31T11:33:32+08:00
title: "使用delve调试golang程序"
description: ""
disqus_identifier: 1485833612031638285
slug: "shi-yong--delve-diao-shi--golang-cheng-xu"
source: "https://segmentfault.com/a/1190000005339023"
tags: 
- delve 
- golang 
topics:
- 编程语言与开发
---

安装 delve
----------

官方的文档已经很全了，参考这里（[github](https://github.com/derekparker/delve/tree/master/Documentation/installation)）安装。Mac
OSX 比较麻烦，需要搞定证书。

断点和变量打印
--------------

delve 装好之后就可以直接在命令行使用 dlv 命令了。

查看可用的命令

    dlv -h

看帮助信息可知：支持用 dlv attach 的方式调试正在运行的进程，支持 dlv
connect 链接到网络端口调试。这里先用最简单的 dlv debug 调试。

    dlv debug main.go

然后会进入调试模式，此时 help 可以看到支持的命令。

    (dlv) b main.go:16 #在 main.go 的第 16 行设置断点。
    (dlv) bp   #查看当前所有断点
    (dlv) c    #运行到下一个断点或者程序结尾
    (dlv) p a  #打印变量 a 的值
    (dlv) n    #单步执行代码
    (dlv) set a=1  #设置变量a 的值

