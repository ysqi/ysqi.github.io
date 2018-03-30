
---
date: 2016-12-31T11:33:59+08:00
title: "Golang包依赖管理工具gb"
description: ""
disqus_identifier: 1485833639885126116
slug: "Golangbao-yi-lai-guan-li-gong-ju-gb"
source: "https://segmentfault.com/a/1190000004346513"
tags: 
- golang 
topics:
- 编程语言与开发
---

原文链接：<http://tabalt.net/blog/golang-package-dependency-management-tool-gb/>

一个Golang项目工程通常由`bin`、`pkg`、`src`三个子目录构成，gb在这个概念的基础上新增了一个`vendor`目录来存放项目依赖的第三方包；一个gb项目的工作目录里包含该项目需要的所有Go代码。

gb项目不放在你的\$GOPATH中，也不需要为你的gb项目设置或修改\$GOPATH。依赖的第三包需要放到`vendor/src`目录中，并使用gb来编译和测试你的项目。

### 安装gb

gb的官网是：[](http://getgb.io/)<http://getgb.io/>，github地址是：[](https://github.com/constabulary/gb/)<https://github.com/constabulary/gb/>。

使用如下命令即可安装gb：

    go get github.com/constabulary/gb/...

安装gb后，会有`gb`和`gb-vendor`两个可执行文件放入你的`$GOPATH/bin`目录中，查看或编辑你的`~/.bash_profile`文件，确保你的`$GOPATH/bin`目录已经加入`$PATH`中：

    export PATH=$PATH:$GOPATH/bin

### 使用gb进行项目开发

我们以一个简单的提供HTTP页面的“Hello
World”程序来学习一下gb的使用。为了体现gb管理第三方包依赖的特性，我们引入一个支持HTTP服务优雅重启的第三方包
[github.com/tabalt/gracehttp](https://github.com/tabalt/gracehttp)。

创建gb项目目录结构：

    cd ~/helloworld
    mkdir -p src/helloworld
    mkdir -p vendor/src

编写“Hello World”程序

    #vim src/helloworld/main.go
    package main

    import (
        "fmt"
        "net/http"

        "github.com/tabalt/gracehttp"
    )

    func main() {
        http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
            fmt.Fprintf(w, "hello world")
        })

        err := gracehttp.ListenAndServe(":8080", nil)
        if err != nil {
            fmt.Println(err)
        }
    }

添加依赖的第三包

    gb vendor fetch github.com/tabalt/gracehttp

目前为止整个项目目录结构如下：

    ./
    |-- src
    |   `-- helloworld
    |       `-- main.go
    `-- vendor
        |-- manifest
        `-- src
            `-- github.com
                `-- tabalt
                    `-- gracehttp
                        |-- README.md
                        |-- connection.go
                        |-- gracehttpdemo
                        |   `-- main.go
                        |-- listener.go
                        `-- server.go

编译执行程序

    gb build helloworld
    ./bin/helloworld

打开一个新终端并执行`curl http://127.0.0.1:8080/`,将会输出：

    hello world

提交所有代码到git仓库

    git init
    git add .
    git commit -am 'init hello world project with gb'
    git add remote -v $your_remote_git_repository
    git push origin master:master

### gb常用命令

在上面的项目开发中，我们用到了两个命令`gb build` 和
`gb vendor`，实际上，`build`是我们之前所说的可执行文件`$GOPATH/bin/gb`包含的，而vendor是gb的一个插件，最终调用的是可执行文件`$GOPATH/bin/gb-vendor`。

可以通过`gb help`命令查看gb支持的更多命令，命令的具体用法可以通过`gb help $command_name`查看，很多gb命令都是在go命令行工具的基础上做的包装，用法也都相似，通过`gb vendor help`可以查看vendor插件具体用法，这里我们简单列举如下：

gb 命令列表

  命令       功能
  ---------- ----------------------
  build      编译包
  doc        显示文档
  env        打印项目的环境变量
  generate   处理源代码生成Go文件
  info       显示项目的信息
  list       显示项目下的所有包
  test       执行测试

gb vendor 功能列表

  参数      功能
  --------- ----------------------------
  fetch     获取一个远程依赖
  update    更新一个本地依赖
  list      每行一个列出所有依赖
  delete    删除一个本地依赖
  purge     清除所有未引用的依赖
  restore   从manifest清单文件还原依赖

原文链接：<http://tabalt.net/blog/golang-package-dependency-management-tool-gb/>

