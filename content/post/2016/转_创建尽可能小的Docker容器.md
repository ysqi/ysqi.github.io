
---
date: 2016-12-31T11:34:48+08:00
title: "创建尽可能小的Docker容器"
description: ""
disqus_identifier: 1485833688325027294
slug: "chuang-jian-jin-ke-neng-xiao-de--Docker-rong-qi"
source: "https://segmentfault.com/a/1190000000628247"
tags: 
- golang 
- dockerfile 
- docker 
categories:
- 编程语言与开发
---

> 注：本文由 [Adriaan de
> Jonge](http://blog.xebia.com/2014/07/04/create-the-smallest-possible-docker-container/#)
> 编写，本文的原文地址为 [Create The Smallest Possible Docker
> Container](http://blog.xebia.com/2014/07/04/create-the-smallest-possible-docker-container/)

当我们在使用 Docker 的时候，你会很快注意到你正在下载很多 MB
作为你的预先配置的容器。一个简单的 Ubuntu 容器很容易超过 200
MB，并且随着在上面安装软件，尺寸在逐渐增大。在某些情况下，你不需要任何事情都使用
Ubuntu 。例如，如果你只是简单的想运行一个 web 服务，使用 GO
编写的，没有必要围绕它使用任何工具。

我一直在寻找尽可能小的容器入手，并且发现了一个：

    docker pull scratch

scratch 镜像是完美的，真正的完美！它简洁，小巧以及快速。它不包含任何
bug，安全泄漏，慢的代码或是技术债务。这是因为它是一个空的镜像。除了一点由
Docker 加入的元数据。事实上，你可以使用如下命令按照 [Docker
文档](https://docs.docker.com/articles/baseimages/#creating-a-simple-base-image-using-scratch)描述的那样创建一个自己的
scratch 镜像。

    tar cv --files-from /dev/null | docker import - scratch

所以这可能就是最小的 Docker 镜像。

或者我们可以说说关于这个的更多东西？比如，你怎样使用 scratch
镜像。这给自己带来了一些挑战。

为 scratch 镜像创建内容
=======================

我们可以在一个空镜像中运行什么？一个没有依赖的可执行程序。你是否有没有依赖的可执行程序？

我过去常常使用 Python，Java 和 Javascript
编写代码。每一个这样的语言/平台都需要一个运行时的安装。最近，我开始涉及
Go（或是 golang 如果你喜欢）平台。看起来 Go
是静态连接的。因此我尝试编译一个简单的 web 服务输出 Hello World
并且运行在 scratch 容器中。下面是这个 Hello World web 服务的代码：

    package main

    import (
        "fmt"
        "net/http"
    )

    func helloHandler(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintln(w, "Hello World from Go in minimal Docker container")
    }

    func main() {
        http.HandleFunc("/", helloHandler)

        fmt.Println("Started, serving at 8080")
        err := http.ListenAndServe(":8080", nil)
        if err != nil {
            panic("ListenAndServe: " + err.Error())
        }
    }

明显地，我不能在 scratch 容器中编译我的 web 服务，因为容器中没有 Go
编译器。正如我在 Mac 上工作，我也无法编译 Linux
的二进制文件一样（实际上，是可以在不同的平台上交叉编译 Go
的源码的，但这会在另外一篇博客中介绍）。

因此，我首先需要一个有 Go 编译器的 Docker 容器。让我们开始：

    docker run -ti google/golang /bin/bash

在这个容器里面，我可以构建一个 Web 服务，通过我已经提交到[一个 GitHub
仓库](https://github.com/adriaandejonge/helloworld)的代码。

    go get github.com/adriaandejonge/helloworld

go get 命令是 go build
命令的变种，运行获取和构建远程的依赖。你可以运行可执行的结果：

    $GOPATH/bin/helloworld

它工作了，但是这不是我们想要的。我们需要 hello world 容器运行在 scratch
容器里面。因此，实际上，我们需要一个 Dockerfile :

    FROM scratch
    ADD bin/helloworld /helloworld
    CMD ["/helloworld"]

然后启动它，不幸的是，我们开始 google/golang 容器的这个方法，
没有办法构建这个 Dockerfile
。因此，首先，我们需要一种方法从这个容器内部访问到 Docker。

从 Docker 内部调用 Docker
=========================

当你使用 Dokcer 时，你迟早会遇到需要从 Docker 内部访问
Docker。可以有多种方法实现它。你可以使用递归和[在 Docker 中运行
Docker](https://github.com/jpetazzo/dind)。尽管如此，这样看起来会很复杂并且导致容器很大。你还可以使用一些额外的命令选项在实例外访问
Docker 服务器：

    docker run -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):$(which docker) -ti google/golang /bin/bash

在你继续前，你重新运行 Go 编译器，由于在重启动过程中 Docker
忘记了我们以前编译过。

    go get github.com/adriaandejonge/helloworld

当我们启动这个容器， `-v` 参数在 Docker 容器中创建一个卷并且允许你从
Docker 的机器提供一个文件作为输入。`/var/run/docker.sock` 是 UNIX
socket，通过这个允许你访问 Docker 服务。 `(which docker)`
部分是一个非常聪明的方法，它提供了一个在 容器中的 Docker
可执行文件的路径，而不是硬编码。尽管如此，当你在 Mac 上通过 boot2docker
使用这个命令的时候需要小心。如果 Docker 的可执行文件与 boot2docker
虚拟机的在不同的位置，将导致不匹配。因此，你或许想使用
`/usr/local/bin/docker` 硬编码的方式替换
`$(which docker)`，如果你运行在不同的系统，`/var/run/docker.sock`
有在不同位置的机会，你需要做相应的调整。

现在你可以在 google/golang 容器的 \$GOPATH 目录使用 Dockerfile
，在这个示例中指向 `/gopath`。实际上，我已经在 github 上检查过了这个
`Dockerfile`，因此，你可以从 Go build 目录复制它到所需的位置，像这样：

    cp $GOPATH/src/github.com/adriaandejonge/helloworld/Dockerfile $GOPATH

你需要复制这个作为二进制的编译文件，现在位于
\$GOPATH/bin，并且它不可能从父目录包含文件当构建一个 Dockerfile
的时候。因此复制后，下一步是：

    docker build -t adejonge/helloworld $GOPATH

所有的都完成以后， Docker 给出如下响应：

    Successfully built 6ff3fd5a381d

允许你运行这个容器：

    docker run -ti --name hellobroken adejonge/helloworld

但是不幸的是， Docker 这次响应如下：

    2014/07/02 17:06:48 no such file or directory

那么到底是怎么回事？我们在 scratch
容器中有可执行的静态链接。难道我们犯了一个错误？

事实证明，Go 不是静态链接库。或者至少不是所有的库。在 Linux
下，我们可以使用 ldd 命令来看到动态链接库：

    ldd $GOPATH/bin/helloworld 

得到如下响应：

    linux-vdso.so.1 => (0x00007fff039fe000)
    libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f61df30f000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f61def84000)
    /lib64/ld-linux-x86-64.so.2 (0x00007f61df530000)

因此，在我们运行我们的 web 服务之前，我需要告诉 go
编译器实际的静态链接。

创建在 Go 中的可执行静态链接
============================

为了创建可执行的静态链接，我们需要告诉 Go 使用 cgo 编译器而不是 go
编译器。命令如下：

    CGO_ENABLED=0 go get -a -ldflags '-s' github.com/adriaandejonge/helloworld

`CGO_ENABLED` 环境变量告诉 Go 使用 cgo 编译器而不是 go 编译器。`-a`
参数告诉 GO 重薪构建所有的依赖。否则的话你将以动态链接依赖结束。最后的
`-ldflags '-s'` 参数是一个非常好的扩展。它大概降低了可执行文件 50%
的文件大小。你也可以不通过 cgo
使用这个。尺寸缩小是去除了调试信息的结果。

为了确定，运行 ldd 命令：

    ldd $GOPATH/bin/helloworld 

返回是：

    not a dynamic executable

你也可以重新运行步骤，围绕着从 scratch 创建 Docker 容器的可执行文件。

    docker build -t adejonge/helloworld $GOPATH

如果一切顺利，Docker 将响应如下：

    Successfully built 6ff3fd5a381d

允许你运行这个容器：

    docker run -ti --name helloworld adejonge/helloworld

响应如下：

    Started, serving at 8080

到目前为止，有许多手动的步骤和很多错误的地方。让我们退出 google/golang
容器并且从周边服务器继续：

    <Press Ctrl-C>
    exit

你可以检查 Docker 容器和镜像存在不存在：

    docker ps -a
    docker images -a

你可以使用如下命令清理：

    docker rm -f helloworld
    docker rmi -f adejonge/helloworld

创建一个 Docker 容器来创建一个 Docker 容器
==========================================

目前为止，我们花了那么多步骤，我们还可以记录在 Dockerfile 中并且 Docker
会为我们做这些工作：

    FROM google/golang
    RUN CGO_ENABLED=0 go get -a -ldflags '-s' github.com/adriaandejonge/helloworld
    RUN cp /gopath/src/github.com/adriaandejonge/helloworld/Dockerfile /gopath
    CMD docker build -t adejonge/helloworld gopath

我在 [一个单独的称为 adriaandejonge/hellobuild 的 GitHub
仓库](https://github.com/adriaandejonge/hellobuild)检查了
Dockerfile。它可以使用下面的命令构建：

    docker build -t adejonge/hellobuild github.com/adriaandejonge/hellobuild

提供 `-t` 参数命名 adejonge/hellobuild
镜像并且它的最新的隐式的标签。这些名字让你以后更容易去除镜像。下一步，你可以使用就像我们在这篇文章前面看到的那样提供一个参数从这个镜像中创建一个容器：

    docker run -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):$(which docker) -ti --name hellobuild adejonge/hellobuild

提供 `--name hellobuild`
参数使得在运行后更容易移除容器。事实上，你可以这样做，因为运行这个命令后，你已经创建了一个
adejonge/helloworld 镜像：

    docker rm -f hellobuild
    docker rmi -f adejonge/hellobuild

现在你可以创建一个基于 adejonge/helloworld 镜像的名为 helloworld
的新容器，就像你以前做的那样：

    docker run -ti --name helloworld adejonge/helloworld

因为所有的这些步骤都是从相同的命令中运行，不需要在 Docker 中打开一个
bash shell 。你可以把这些步骤添加进一个 bash
脚本，自动运行它，为了使你方便，[我已经把这些脚本加入了 hellobuild
GitHub
仓库](https://github.com/adriaandejonge/hellobuild/tree/master/scripts)。

另外，如果你想尝试一个尽可能小的容器，但是又不想遵循博客中的步骤，你可以使用我检入进
[Docker Hub
repository](https://registry.hub.docker.com/u/adejonge/helloworld/)
的预先构建好的镜像。

    docker pull adejonge/helloworld

使用 docker images -a ，你可以看到大小是
3.6MB。当然，如果你成功创建一个比我使用 Go 编写的 web
服务还小的可执行文件，你可以使得它更小。使用 C
语言或者是汇编，你可以这样做到。尽管如此，你不可能使得它比 scratch
镜像还小

**扩展阅读**

-   [OPTIMIZING DOCKER
    IMAGES](http://www.centurylinklabs.com/optimizing-docker-images/)


