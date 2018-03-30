
---
date: 2016-12-31T11:33:52+08:00
title: "Go初体验｜Mac上安装Go"
description: ""
disqus_identifier: 1485833632102480516
slug: "Gochu-ti-yan-｜Macshang-an-zhuang-Go"
source: "https://segmentfault.com/a/1190000004851499"
tags: 
- golang 
topics:
- 编程语言与开发
---

笔记

从国内镜像下载安装包：<http://www.golangtc.com/download>

进入配置文件修改环境变量

    vim ~/.bashrc

编辑`GOROOT`,`GOPATH`,`PATH`

    export GOROOT=/usr/local/go
    export PATH=$PATH:$GOROOT/bin
    export GOPATH=/data/www/go

解释

`GOROOT` 表示 Go 在你的电脑上的安装位置，它的值一般都是 \$HOME/go\
`GOARCH` 表示目标机器的处理器架构，它的值可以是 386，amd64 或 arm\
`GOOS` 表示目标机器的操作系统，它的值可以是 darwin，freebsd，linux 或
windows\
`GOBIN` \$GOBIN 表示编译器和链接器的安装位置，默认是 \$GOROOT/bin\
`GOPATH`
表示工作路径，允许包含多个目录。当有多个目录时，请注意分隔符，多个目录的时候Windows是分号，Linux系统是冒号，当有多个GOPATH时，默认会将go
get命令的内容放在第一个目录下。\
`GOPATH`约定有三个子目录：

-   src 存放源代码（比如：.go .c .h .s等）

-   pkg 编译后生成的文件（比如：.a）

-   bin
    编译后生成的可执行文件（为了方便，可以把此目录加入到`$PATH`变量中，如果有多个gopath，那么使用\${GOPATH//://bin:}/bin添加所有的bin目录），很多Go命令都依赖于此变量，例如`go get`命令会将获取到的包放到GOPATH中。

常用commands：

    $ go version
    $ go help
    $ go run file.go
    $ go build file.go
    $ ./file

参考：

1.  [https://github.com/Unknwon/the-way-to-go...](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/02.3.md)

2.  <http://www.tuicool.com/articles/Fv6zUfE>

3.  [https://www.shiyanlou.com/courses/runnin...](https://www.shiyanlou.com/courses/running/54)



