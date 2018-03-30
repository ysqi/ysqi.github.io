
---
date: 2016-12-31T11:34:36+08:00
title: "【新功能】GitCafe已经支持Go语言Package托管"
description: ""
disqus_identifier: 1485833676239480466
slug: "【xin-gong-neng-】GitCafe-yi-jing-zhi-chi--Go-yu-yan--Package-tuo-guan"
source: "https://segmentfault.com/a/1190000002556796"
tags: 
- 代码托管 
- golang 
- git 
topics:
- 编程语言与开发
---

之前有用户希望 GitCafe 可以支持 go get，使其能够直接导入 GitCafe
上托管的代码。

现在，我们想要告诉各位一个好消息，为了方便更多的开发者，任何托管在
GitCafe 上的 Go 语言 package 都可以直接在项目中使用，通过 go get
工具安装和更新。

例如，我们可以在本机创建一个 hello\_world.go 文件，内容如下：
```Go
    package main

    import "gitcafe.com/rainux/go-hello"

    func main() {
        hello.World()
    }
```
执行 go get -d 命令可以将 GitCafe 上托管的 go-hello 项目源代码安装到
\$GOPATH 中；执行 go run hello\_world.go 可以输出经典的 "Hello world!"
信息。

如下所示：\
\
希望各位开发者都可以在 GitCafe 更加方便地进行开发，现在就开始吧！

Happy Go Programming!

#### Tips:

Go （也叫 "golang"）是一款由 Google
最初开发的编程语言。它自诞生就有几个设计原则：简单性、安全性和速度。Go
语言发行版拥有各种调试、测试、调优和代码审查工具。

go get 是 golang 默认的包管理工具，它支持通过 git/mercurial
等版本控制工具远程导入包

