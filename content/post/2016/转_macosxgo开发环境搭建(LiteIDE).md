
---
date: 2016-12-31T11:34:06+08:00
title: "macosxgo开发环境搭建(LiteIDE)"
description: ""
disqus_identifier: 1485833646029235210
slug: "mac-osx-gokai-fa-huan-jing-da-jian-(LiteIDE)"
source: "https://segmentfault.com/a/1190000004008942"
tags: 
- liteide 
- golang 
categories:
- 编程语言与开发
---

### 1.下载go

> 官网下载地址（需要翻墙）\
> [golang](https://golang.org/dl/)

> 中国镜像网站下载\
> [golangtc](http://www.golangtc.com/download)

下载go1.5.1.darwin-amd64.tar.gz包，解压得到一个go目录，把go目录移动到你想要的路径下，我这里是放在`~/Documents/go`这个路径下的

### 2.配置环境变量

打开终端使用命令`vim .bash_profile `打开配置文件并修改保存，添加如下语句

     #GOPATH
     export GOPATH=~/Applications/Go
     #GOROOT
     export GOROOT=~/Documents/go
     #PATH
     export PATH=$GOROOT/bin:$GOPATH/bin:$PATH

`GOROOT`变量是下载解压的go目录的路径，`GOPATH`这个变量是必须设置的，而且不能和go的安装目录及`GOROOT`一样，这个目录用来存放Go源码，Go的可运行文件，以及相应的编译之后的包文件。所以这个目录下面有三个子目录：src、bin、pkg。这里我在`~/Applications/`下新建一个`Go`目录，并在`Go`目录中新建scr、bin、pkg三个目录。

> 更深入的了解go的环境变量或者学习可看下面这本书\
> [build-web-application-with-golang](https://github.com/astaxie/build-web-application-with-golang)

配置完成保存退出后运行如下命令使环境变量生效\
`source .bash_profile`

### 3.下载安装配置LiteIDE

> 下载地址\
> [LiteIDE Download](http://golangtc.com/download/liteide)

安装好后打开LiteIDE
编辑当前环境，这里的配置和前面的go环境变量配置一样，其实前面的环境变量配置可以忽略，到这里再配置

> `查看->编辑当前环境` 打开

配置如下

    # native compiler drawin amd64

    GOROOT=$HOME/Documents/go
    GOPATH=$HOME/Applications/Go
    GOBIN=
    GOARCH=amd64
    GOOS=darwin
    CGO_ENABLED=1

    PATH=$GOROOT/bin:$GOPATH/bin:$PATH

    LITEIDE_GDB=/usr/local/bin/gdb
    LITEIDE_MAKE=make
    LITEIDE_TERM=/usr/bin/open
    LITEIDE_TERMARGS=-a Terminal
    LITEIDE_EXEC=/usr/X11R6/bin/xterm
    LITEIDE_EXECOPT=-e

保存退出后就搭建完成了

### 4.Hello world

在`$GOPATH/src/`下创建目录`hello`使用LiteIDE打开目录，创建文件`main.go`

    package main

    import "fmt"

    func main(){
        fmt.Println("Hello world!")
    }

点击上面的BR 按钮就可运行 build and run

