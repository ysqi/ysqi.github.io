
---
date: 2017-02-08T13:41:19+08:00
title: "GB(一)"
description: ""
disqus_identifier: 1486532479430240120
slug: "GB(yi-)"
source: "https://segmentfault.com/a/1190000008264452"
tags: 
- golang 
topics:
- 编程语言与开发
---

> **[gb](https://getgb.io/)** go语言基于项目的编译工具

1. 安装
=======

### 1.1 约束

[gb](https://getgb.io/) 依赖Go1.4以上版本

### 1.2 安装

通过以下命令安装

    go get github.com/constabulary/gb...

### 1.3 升级

[gb](https://getgb.io/) 依然处于开发状态，通过以下命令升级到最新版本

    go get -u github.com/constabulary/gb/...

### 1.4 多版本go的情况

对每一个go版本都安装[gb](https://getgb.io/)

### 1.5 注意

安装完毕后的gb命令和oh-my-zsh配置的 `git branch` 简写命令有冲突，采用
\~/.zshrc中

    unalias gb

来屏蔽

2. 项目
=======

[gb](https://getgb.io/)基于项目。一个[gb](https://getgb.io/)工程为一个编译单元，每个[gb](https://getgb.io/)工程目录含有一个`src/`子目录，没有配置文件的设置，以下文档我们统称工程的目录为`$PROJECT`

### 2.1 自己的代码，第三方的代码

[gb](https://getgb.io/)项目区分自己的代码和依赖的第三方代码。[gb](https://getgb.io/)项目内，自己的代码放在

    $PROJECT/src/

第三方代码放在

    $PROJECT/vendor/src/

### 2.2 项目不在\$GOPATH下进行配置

[gb](https://getgb.io/)项目不会跟`$GOPATH`有关系，\
[gb](https://getgb.io/)也不会采用`go get`来下载管理依赖；依赖的第三方库代码都应放在`$PROJECT/vendor/src/`
目录下\
[gb](https://getgb.io/)项目也可以用`go get`来获取，但不能由`go tools`工具来构建，因为[gb](https://getgb.io/)项目不遵循`go get`的约定

### 2.3 创建项目

创建一个[gb](https://getgb.io/)项目也就是创建一个普通的文件目录：

    % mkdir -p $HOME/code/demo-project

这个目录将作为[gb](https://getgb.io/)项目的根目录，现在创建`src/`子目录来存放你自己的项目代码：

    % mkdir -p $PROJECT/src
    % tree $PROJECT
    /home/dfc/code/demo-project
    └── src

### 2.4 创建包

注意:
[gb](https://getgb.io/)不会编译`$PROJECT/src/`下的代码，也不会编译根目录下的代码，你必须将代码放在一个package内，让我们来创建一个包：

    % mkdir -p $PROJECT/src/hello
    % tree $PROJECT
    /home/dfc/code/demo-project
    └── src
        └── hello
            └── hello.go

我们看一下hello.go文件:

    package main
     
    import "fmt"
     
    func main() {
        fmt.Println("Hello gb")
    }

### 2.5 编译

注意：采用[gb](https://getgb.io/)自己的编译命令：

    % gb build all
    hello
    % bin/hello
    Hello gb
    % tree $PROJECT
    /home/dfc/code/demo-project
    ├── bin
        └── hello
    └── src
        └── hello
            └── hello.go

### 2.6 版本控制

注意：一般不提交`$PROJECT/pkg`和`$PROJECT/bin`下的内容，只提交`$PROJECT/src/`下的代码

* * * * *

