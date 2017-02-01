
---
date: 2016-12-31T11:34:02+08:00
title: "installgolang开发环境withwin64"
description: ""
disqus_identifier: 1485833642953161880
slug: "install-golang-kai-fa-huan-jing--with-win-64"
source: "https://segmentfault.com/a/1190000004209996"
tags: 
- go入门 
- go语言 
- go开发环境 
- 开发环境 
- golang 
topics:
- 编程语言与开发
---

**1.下载 并且 安装 Go安装包**

百度网盘上传了最新GO版本，供大家下载：<http://pan.baidu.com/s/1bjg9zg>

===========================================================\
注意：千万不要在安装路径中出现中文。否则之后将无法正常使用Go语言开发工具

安装说明的链接：（可能需要翻墙）\
<https://code.google.com/p/golang-china/wiki/Install>\

下载Go安装包的链接：\
<http://pan.baidu.com/s/1bjg9zg>

**2.配置环境变量**

(1). 新建 变量名：GOBIN 变量值 ：c:\\go\\bin

(2). 新建 变量名：GOARCH 变量值：amd64

(3). 新建 变量名：GOOS 变量值：windows

(4). 新建 变量名： GOROOT 变量值：c:\\go

(5). 编辑 Path 在Path的变量值的最后加上 %GOBIN% *(默认已加)*

如果是msi安装文件，Go语言的环境变量会自动设置好。如果后面的测试无法通过，可以重新设置环境变量。

**3.测试安装是否成功**

打开Windows中的命令提示符（cmd.exe）执行命令：go version 或者 go help\
正常情况下会显示：\

**4.访问Go安装包中的文档**\
打开Windows中的命令提示符（cmd.exe）执行命令: godoc -http=:8080

**5.输出“Hello World!”**

1）在C盘新建一个文件：hello.go

    2）输入或者直接复制粘贴代码：
    package main

    import "fmt"

    func main(){

    fmt.Printf("Hello Word!\n");

    }

    注意：大括号一定要这么写，这是因为go在语法中加入一些代码规范，按照下面这样写是错误的：
    func main()
    {
    fmt.Printf("Hello Word!\n");
    }

    打开Windows中的命令提示符（cmd.exe）执行命令：
    go build -o C:\hello.exe C:\hello.go
    或者
    go build C:\test.go

(注意：上面一条指定了输出的exe文件存在C:\\hello.exe,而下面一条会在当前路径下生成hello.exe(可能会不是C:\\hello.go的位置))\
编译成功后，会在c盘生成一个hello.exe文件\
4）执行hello.exe，在命令提示符中执行命令：\
hello.exe\
将会输出：\
Hello Word！\
我是在D盘创建的hello.go文件的，所以略有不同。

<http://shang.qq.com/wpa/qunwpa?idkey=912ee9f1c4d48fd64877a62ce321ca041ff6780dee63a5ccaa07241719ea5be9>多语言/高并发研究群

