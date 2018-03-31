
---
date: 2016-12-31T11:33:11+08:00
title: "Golang之工程结构"
description: ""
disqus_identifier: 1485833591547400465
slug: "Golang-zhi--gong-cheng-jie-gou"
source: "https://segmentfault.com/a/1190000006726883"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

综述
----

-   典型地, Go 将所有 Go 代码都存放到单一的
    workspace 中(存放在单一的一个目录中).

-   一个 workspace 包含多个版本控制仓库(version control repositories,
    例如 Git), 即一个 workspace 包含多个 go 工程.

-   每个版本控制仓库(工程) 包含一个或多个包

-   一个包包含一个或多个 Go 源文件

-   包的路径决定了它的 **import** 路径.

关于 workspace
--------------

一个 workspace 就是一个目录的层次结构,
并且在顶层目录中包含如下三个子目录:

-   src, 包含多个版本控制仓库(即 工程).

-   pkg, 包含编译后的 package 对象

-   bin, 包含编译后的可执行文件.

一个典型的 workspace 如下:

    bin/
        hello                          # command executable
        outyet                         # command executable
    pkg/
        linux_amd64/
            github.com/golang/example/
                stringutil.a           # package object
    src/
        github.com/golang/example/
            .git/                      # Git repository metadata
            hello/
                hello.go               # command source
            outyet/
                main.go                # command source
                main_test.go           # test source
            stringutil/
                reverse.go             # package source
                reverse_test.go        # test source
        golang.org/x/image/
            .git/                      # Git repository metadata
            bmp/
                reader.go              # package source
                writer.go              # package source
        ... (many more repositories and packages omitted) ...

上面的 workspace 包含了两个仓库(example 和 image). example 仓库中有两个
command(hello 和 outyet) 和一个库(stringutil). image 仓库包含 bmp
包和其他的文件.

关于 GOPATH
-----------

GOPATH 环境变量指定了 workspace 的目录,
我们仅仅指定这个环境变量就可以进行 Go 的开发了.

关于 import path
----------------

一个 import path 是一个唯一标识一个包的字符串. 一个包的 import path
由这个包在 workspace 中的路径决定, 或者与这个包所在的远程仓库有关.\
如果代码是使用代码仓库管理(例如 Github), 那么 Go
建议使用仓库名作为包的基路径. 例如 github.com/yongshun 是包的基路径,
我们如果需要创建一个名为 hello 的工程, 那么它的包路径就是
github.com/yongshun/hello

关于包名
--------

在一个 go 文件中, 第一行代码必须是一个包定义:

    package name

`注意, 根据 GO 规定, 包名是一个包的 import path 的最后一个元素, 即如果 import path 是 "com/xys/hello", 则包名必须是 hello.`\
`可执行的 command 的包名必须是 main.`\
`即使是链接到同一个 binary 中, Go 也不要求所有的包名是唯一的, 即链接到一个 binary 中的包名可以是重复的, 只要包对应的 import path 是唯一的即可.`

