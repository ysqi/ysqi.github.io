
---
date: 2016-12-31T11:34:35+08:00
title: "Go"
description: ""
disqus_identifier: 1485833675111525811
slug: "Go"
source: "https://segmentfault.com/a/1190000002623275"
tags: 
- golang 
categories:
- 编程语言与开发
---

Go语言介绍
----------

1.  官方

    1.  网站：[](http://golang.org)<http://golang.org>
    2.  源码：[](http://github.com/golang/go)<http://github.com/golang/go>

2.  Go语言特点

    1.  简单易学，类似C语言的风格
    2.  内置了goroutine机制，语言层面支持并发
    3.  Go目前已经内置了大量的标准库
    4.  跨平台编译
    5.  内嵌C支持，可利用丰富的C语言库

3.  学习社区

    1.  官网 [](http://golang.org/project/)<http://golang.org/project/>
    2.  Golang中文社区
        [](http://studygolang.com/)<http://studygolang.com/>
    3.  Golang中国 [](http://golangtc.com)<http://golangtc.com>

4.  开源项目

    1.  Docker [](http://www.docker.com/)<http://www.docker.com/>\
        Docker是一个开源的应用容器引擎
    2.  etcd
        [](https://github.com/coreos/etcd/)<https://github.com/coreos/etcd/>\
        etcd是一个高可用的Key/Value存储系统，主要用于分享配置和服务发现
    3.  nsq
        [](https://github.com/bitly/nsq/)<https://github.com/bitly/nsq/>\
        实时分布式的消息平台
    4.  更多开源项目：[](https://github.com/golang/go/wiki/Projects/)<https://github.com/golang/go/wiki/Projects/>

5.  学习资料推荐

    1.  官方文档\
        \
        -   官网：[](http://golang.org/doc/)<http://golang.org/doc/>
        -   国内访问：[](http://godoc.golangtc.com/doc/)<http://godoc.golangtc.com/doc/>

    2.  开源书籍《Go Web 编程》\
        [](https://github.com/astaxie/build-web-application-with-golang)<https://github.com/astaxie/build-web-application-with-golang>
    3.  go语言的中文翻译\
        [](http://github.com/Go-zh/go)<http://github.com/Go-zh/go>
    4.  GO开发者对GO初学者建议[](http://segmentfault.com/a/1190000000654351)<http://segmentfault.com/a/1190000000654351>

Go安装与配置
------------

1.  Go中的三个环境变量\
    \
    1.  GOROOT\
        Go语言安装的路径，如MAC下：/usr/local/go，类似JAVA中的JAVA\_HOME
    2.  GOPATH\
        GOPATH表示包所在的地址，可以设置多个。\
        假设：GOPATH=\~/go1:\~/go2，GOROOT=/usr/local/go，在代码中引用了包：github.com/bitly/nsq/util\
        那么: Go在编译时会按先后次序到以下目录中寻找源码：\
        \~/go1/src/github.com/bitly/nsq/util\
        \~/go2/src/github.com/bitly/nsq/util\
        /usr/local/go/src/github.com/bitly/nsq/util
    3.  PATH\
        可执行实例的路径

2.  Go命令\
    \
    -   学习文档
        [](https://github.com/hyper-carrot/go_command_tutorial)<https://github.com/hyper-carrot/go_command_tutorial>
    -   go 显示命令相关帮助信息
    -   go build 编译包和依赖，会在执行命令时所在目录生成可执行文件
    -   go clean 清理编译生成的文件
    -   go env 显示go环境变量
    -   go fmt 格式化代码
    -   go install 编译和安装包、依赖
    -   go run 编译并运行go程序
    -   go get 获取或更新代码包及其依赖，并对他们进行编译和安装
    -   go test 运行测试代码

3.  Go Web开发框架\
    \
    1.  Beego [](http://beego.me/)<http://beego.me/>\
        MVC框架，作者为中国人，框架中中文文档丰富，用户群体大，便于交流。学习难度低。
    2.  Revel [](http://revel.github.io/)<http://revel.github.io/>\
        思路来自Java的Play Framework，相对Beego难一点。
    3.  Martini
        [](http://martini.codegangsta.io/)<http://martini.codegangsta.io/>\
        简单灵活，大量使用反射，初学不易上手。



