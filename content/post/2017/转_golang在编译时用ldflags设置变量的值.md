
---
date: 2017-02-17T08:17:12+08:00
title: "golang在编译时用ldflags设置变量的值"
description: ""
disqus_identifier: 1487290632810219765
slug: "golangzai-bian-yi-shi-yong-ldflagsshe-zhi-bian-liang-de-zhi"
source: "https://segmentfault.com/a/1190000008323048"
tags: 
- golang 
topics:
- 编程语言与开发
---

我们经常会在一些程序的输出中看到程序版本、编译时间、Git的commit
id等信息，比如docker

    ming@vultr:~$ docker version
    Client:
     Version:      1.12.5
     API version:  1.24
     Go version:   go1.6.4
     Git commit:   7392c3b
     Built:        Fri Dec 16 02:42:17 2016
     OS/Arch:      linux/amd64
    ...

我们可以提供一个配置文件`version.conf`，程序运行时从`version.conf`取得这些信息进行显示。但是在部署程序时，除了二进制文件还需要额外的配置文件，不是很方便。
\
或者将这些信息写入代码中，这样不需要额外的`version.conf`，但要在每次编译时修改代码文件，也够麻烦的了。
\
有一种更好的办法是在编译时使用参数[-ldflags -X
importpath.name=value](https://golang.org/cmd/link/)，官方解释如下

> -X importpath.name=value \
> Set the value of the string variable in importpath named name to
> value. \
> Note that before Go 1.5 this option took two separate arguments. \
> Now it takes one argument split on the first = sign.

以下面代码说明

    package main

    import "fmt"

    var (
        VERSION    string
        BUILD_TIME string
        GO_VERSION string
    )

    func main() {
        fmt.Printf("%s\n%s\n%s\n", VERSION, BUILD_TIME, GO_VERSION)
    }

用如下命令编译，注意**因为`date`和`go version`的输出有空格，所以`main.BUILD_TIME`和`main.GO_VERSION`必须使用引号括起来**

    go build -ldflags "-X main.VERSION=1.0.0 -X 'main.BUILD_TIME=`date`' -X 'main.GO_VERSION=`go version`'"

编译成功后运行程序，结果如下

    ming@ubuntu:~/go_workspace/src/test$ ./test 
    1.0.0
    Sun Feb 12 00:13:27 CST 2017
    go version go1.7.5 linux/amd64

