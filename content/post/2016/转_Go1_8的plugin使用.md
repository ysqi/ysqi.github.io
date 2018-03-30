
---
date: 2016-12-31T11:32:44+08:00
title: "Go1_8的plugin使用"
description: ""
disqus_identifier: 1485833564043144573
slug: "Go-1_8de-pluginshi-yong"
source: "https://segmentfault.com/a/1190000007713877"
tags: 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

Go
1.8为我们提供了一个创建共享库的新工具，称为Plugins！让我们来创建和使用一个插件。
目前的插件只能在Linux和Darwin上工作。

安装1.8beta1,不做说明.

创建一个插件方法到aplugin.go:

    package main

    func Add(x, y int) int {
        return x+y
    }

    func Subtract(x, y int) int {
        return x-y
    }

然后构建插件:

运行下面命令构建插件:

    go build -buildmode=plugin

构建指定文件插件aplugin.go到aplugin.so:

    go build -buildmode=plugin -o aplugin.so aplugin.go

加载插件:

    p, _ := plugin.Open("./aplugin.so")
    //p, err := plugin.Open("./aplugin.so")

call插件:

    add, _ := p.Lookup("Add")
    sub, _ := p.Lookup("Subtract")

使用插件:

    sum := add.(func(int, int) int)(11, 2)
    fmt.Println(sum)
    subt := sub.(func(int, int) int)(11, 2)
    fmt.Println(subt)

另外源码测试中有:

    go build -buildmode=c-shared

应该可以支持c语言构建插件

