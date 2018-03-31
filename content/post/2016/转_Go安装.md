
---
date: 2016-12-31T11:34:49+08:00
title: "Go安装"
description: ""
disqus_identifier: 1485833689653270372
slug: "Goan-zhuang"
source: "https://segmentfault.com/a/1190000000592770"
tags: 
- golang 
categories:
- 编程语言与开发
---

**Go官网**

    http://golang.org/

**下载地址**

    http://golang.org/dl/

**安装环境**

    32位系统 - go1.3.linux-386.tar.gz

    64位系统 - go1.3.linux-amd64.tar.gz

假定你想要安装Go的目录为 `$GO_INSTALL_DIR`

    tar zxvf go1.0.3.linux-amd64.tar.gz -C $GO_INSTALL_DIR

    vim $HOME/.profile  //或者修改/etc/profile

    加入两行：

    export GOROOT=$GO_INSTALL_DIR/go    // Go安装目录

    export PATH=$PATH:$GOROOT/bin       // 环境变量

    source $HOME/.profile

GOROOT只有在自定义安装目录时，才需要设置。

**执行命令**

    [root@localhost] go version

    >> go version go1.3 linux/amd64


    [root@localhost] go

> Go is a tool for managing Go source code.
>
> Usage:
>
> go command \[arguments\]
>
> The commands are:
>
>     build       compile packages and dependencies
>     clean       remove object files
>     env         print Go environment information
>     fix         run go tool fix on packages
>     fmt         run gofmt on package sources
>     get         download and install packages and dependencies
>     install     compile and install packages and dependencies
>     list        list packages
>     run         compile and run Go program
>     test        test packages
>     tool        run specified go tool
>     version     print Go version
>     vet         run go tool vet on packages
>
> Use "go help \[command\]" for more information about a command.
>
> Additional help categories:
>
>     c           calling between Go and C
>     filetype    file types
>     gopath      GOPATH environment variable
>     importpath  import path syntax
>     packages    description of package lists
>     testflag    description of testing flags
>     testfunc    description of testing functions
>
> Use "go help \[topic\]" for more information about that topic.

**Windows下安装**

**安装环境**

    32位系统 - go1.3.windows-386.zip
    64位系统 - go1.3.windows-amd64.zip

假定你想安装的目录为：`C:\Go`

**环境变量**

这里，我们将设置一下环境变量

右键点击`计算机`-&gt; `属性`-&gt; `高级系统设置`-&gt;`环境变量`

`用户变量`：针对当前用户有效。\
`系统变量`：针对所有用户有效。

新建变量名：`GOROOT`。变量值：`C:\Go`。\
新建变量名：`GOPATH`。变量值：`D:\goroot`。\
编辑变量`Path`：变量值最后加入：`%GOROOT%\bin`。

**运行**

开启CMD命令行窗口。

安装成功！

**运行Go程序**

1、创建脚本：`test.go`

    package main

    import "fmt"

    func main() {
        fmt.Printf("hello, world\n")
    }

2、执行脚本

    go run hello.go

3、程序输出

    hello, world

