
---
date: 2016-12-31T11:32:41+08:00
title: "GolanggRPC实践连载一gRPC介绍与安装"
description: ""
disqus_identifier: 1485833561652106745
slug: "Golang-gRPCshi-jian--lian-zai-yi--gRPCjie-shao-yu-an-zhuang"
source: "https://segmentfault.com/a/1190000007880647"
tags: 
- golang 
- grpc 
topics:
- 编程语言与开发
---

gRPC介绍与安装
==============

> A high performance, open source, general RPC framework that puts
> mobile and HTTP/2 first. ——[gRPC Website](http://www.grpc.io/)

[gRPC](http://www.grpc.io/)
是一个高性能、开源、通用的RPC框架，由Google推出，基于[HTTP/2](https://http2.github.io/)协议标准设计开发，默认采用[Protocol
Buffers](https://developers.google.com/protocol-buffers/)数据序列化协议，支持多种开发语言。gRPC提供了一种简单的方法来精确的定义服务，并且为客户端和服务端自动生成可靠的功能库。

主要特性
--------

-   强大的IDL\
    gRPC使用ProtoBuf来定义服务，ProtoBuf是由Google开发的一种数据序列化协议（类似于XML、JSON、hessian）。ProtoBuf能够将数据进行序列化，并广泛应用在数据存储、通信协议等方面。

-   多语言支持\
    gRPC支持多种语言，并能够基于语言自动生成客户端和服务端功能库。目前已提供了C版本grpc、Java版本grpc-java
    和
    Go版本grpc-go，其它语言的版本正在积极开发中，其中，grpc支持C、C++、Node.js、Python、Ruby、Objective-C、PHP和C\#等语言，grpc-java已经支持Android开发。

-   HTTP/2\
    gRPC基于HTTP/2标准设计，所以相对于其他RPC框架，gRPC带来了更多强大功能，如双向流、头部压缩、多复用请求等。这些功能给移动设备带来重大益处，如节省带宽、降低TCP链接次数、节省CPU使用和延长电池寿命等。同时，gRPC还能够提高了云端服务和Web应用的性能。gRPC既能够在客户端应用，也能够在服务器端应用，从而以透明的方式实现客户端和服务器端的通信和简化通信系统的构建。

在gRPC客户端可以直接调用不同服务器上的远程程序，使用姿势看起来就像调用本地程序一样，很容易去构建分布式应用和服务。和很多RPC系统一样，服务端负责实现定义好的接口并处理客户端的请求，客户端根据接口描述直接调用需要的服务。客户端和服务端可以分别使用gRPC支持的不同语言实现。

grpc库与protobuf
----------------

默认需要单独编译安装grpc与protobuf编译器，[grpc/homebrew-grpc](https://github.com/grpc/homebrew-grpc)项目提供了快速安装脚本，可以直接安装grpc、protobuf编译器及其它语言编译需要的插件。go语言插件需要独立安装。

准备工作
--------

-   Linux系统安装 [linuxbrew](https://github.com/Homebrew/linuxbrew)

-   MAC OS X系统安装 [homebrew](http://brew.sh/)

安装
----

脚本安装

    curl -fsSL https://goo.gl/getgrpc | bash -s -- --with-plugins

或者直接使用brew安装：

    brew tap grpc/grpc
    brew install --with-plugins grpc

安装结果：

    /usr/local/bin

    protoc
    grpc_cpp_plugin
    grpc_node_plugin
    grpc_python_plugin
    grpc_csharp_plugin
    grpc_objective_c_plugin
    grpc_ruby_plugin

这种安装方式会安装gRPc
C/C++库和其它支持语言的proto插件和protobuf编译器。如果用不到这些语言，可以只安装protobuf就好了，参考项目：[protobuf](https://github.com/google/protobuf)。java和go支持有独立项目[grpc-java](https://github.com/grpc/grpc-java/)和[grpc-go](https://github.com/grpc/grpc-go/)。

Golang protobuf插件
-------------------

项目地址：[golang/protobuf](https://github.com/golang/protobuf)

安装：

-   要求golang版本 &gt; 1.4

-   使用前要求安装protocol buffer编译器

运行：

    go get -u github.com/golang/protobuf/{proto,protoc-gen-go}

编译后会安装`protoc-gen-go`到`$GOBIN`目录, 默认在 `$GOPATH/bin`.
该目录必须在系统的环境变量`$PATH`中，这样在编译.proto文件时protocol编译器才能找到插件。

grpc-go
-------

    # 如果需要翻墙自己解决吧
    go get -u google.golang.org/grpc

编译器使用
----------

使用`protoc`命令编译`.proto`文件,不同语言支持需要指定输出参数，如：

    protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --javanano_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto

这里详细介绍golang的编译姿势:

-   `-I`
    参数：指定import路径，可以指定多个`-I`参数，编译时按顺序查找，不指定时默认查找当前目录

-   `--go_out` ：golang编译支持，支持以下参数

    -   `plugins=plugin1+plugin2` -
        指定插件，目前只支持grpc，即：`plugins=grpc`

    -   `M` 参数 -
        指定导入的.proto文件路径编译后对应的golang包名(不指定本参数默认就是`.proto`文件中`import`语句的路径)

    -   `import_prefix=xxx` -
        为所有`import`路径添加前缀，主要用于编译子目录内的多个proto文件，这个参数按理说很有用，尤其适用替代一些情况时的`M`参数，但是实际使用时有个蛋疼的问题导致并不能达到我们预想的效果，自己尝试看看吧

    -   `import_path=foo/bar` -
        用于指定未声明`package`或`go_package`的文件的包名，最右面的斜线前的字符会被忽略

    -   末尾 `:编译文件路径  .proto文件路径(支持通配符)`

完整示例：

    protoc -I . --go_out=plugins=grpc,Mfoo/bar.proto=bar,import_prefix=foo/,import_path=foo/bar:. ./*.proto

参考
----

### 本系列示例代码

-   [go-grpc-example](https://github.com/Jergoo/go-grpc-example)

### 相关文档：

-   [grpc官方文档](http://www.grpc.io/docs)

-   [grpc中文文档](http://doc.oschina.net/grpc)

-   [protobuf官方文档](https://developers.google.com/protocol-buffers/)

### 相关项目

-   [grpc/grpc](https://github.com/grpc/grpc)

-   [grpc-go](https://github.com/grpc/grpc-go/)

-   [grpc-java](https://github.com/grpc/grpc-java)

-   [google/protobuf](https://github.com/google/protobuf)

-   [golang/protobuf](https://github.com/golang/protobuf)

-   [grpc-middleware](https://github.com/mwitkow/go-grpc-middleware)

-   [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)



