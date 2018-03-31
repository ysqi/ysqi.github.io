
---
date: 2016-12-31T11:34:17+08:00
title: "多进程管理工具:goreman"
description: ""
disqus_identifier: 1485833657191530455
slug: "duo-jin-cheng-guan-li-gong-ju-:goreman"
source: "https://segmentfault.com/a/1190000003778084"
tags: 
- foreman 
- golang 
categories:
- 编程语言与开发
---

Linux下多进程管理工具对开发和运维都很有用，常见的功能全面的主流工具主要有monit、supervisor。不过开发中使用则推荐轻量级小工具[goreman](https://github.com/mattn/goreman)。

goreman是对Ruby下广泛使用的foreman的重写，毕竟基于golang的工具简单易用多了。顺便提一句：goreman的作者是mattn，在golang社区挺活跃的日本的一名程序员。foreman原作者也实现了一个golang版：forego，不过没有goreman好用，举个例子：coreos的etcd就是使用的goreman来一键启停单机版的etcd集群。

安装
----

go工具安装都非常简单：

    go get github.com/mattn/goreman
    goreman help

当然，记得先把GOPATH、GOROOT环境变量配置好，并记得把\$GOPATH/bin添加到\$PATH

使用
----

由于是小工具，参考goreman help基本就足够了。简单的使用步骤：

1.  新建一个Procfile文件，如果改名则需要goreman -f指定。

2.  在包含Procfile的目录下执行：goreman start

3.  关闭时直接ctrl-c推出，goreman会自动把所有启动的进程都shut down

举例
----

以[Apache
kafka](http://kafka.apache.org/)的使用为例，了解的朋友应该知道，kafka使用时通常需要启动两个进程：一个zookeeper，一个kafka
broker，因此可以编写一个kafka开发环境的Procfile：

    zookeeper: bash ~/tool/kafka_2.11-0.8.2.1/bin/zookeeper-server-start.sh config/zookeeper.properties
    broker: bash ~/tool/kafka_2.11-0.8.2.1/bin/kafka-server-start.sh config/server.properties

然后执行goreman start，可以看到不同颜色区分的zookeeper、kafka
broker进程的启动日志：

    11:04:10 zookeeper | Starting zookeeper on port 5000
    11:04:10    broker | Starting broker on port 5001
    ...

关闭时，直接ctrl-c，则两个bash进程也会被自动关闭。

高级用法
--------

上述是最简单的使用场景：直接使用goreman
start，不过有个缺点，即goreman绑定到了当前的session，而且不能灵活控制多个进程启停以及顺序。而实际开发过程中，通常需要经常单独启停某个正在开发的模块相关的进程，比如上面例子中的kafka-broker，而Zookeeper通常不需要频繁启停。

可以使用更高级的goreman run命令来实现，如：

    # 先启动Zookeeper
    goreman run start zookeeper
    # 然后启动kafka
    goreman run start broker
    # 查看进程状态
    goreman run status
    # 停止broker进程
    goreman run stop broker
    # 重启broker进程
    goreman run restart broker

总结
----

多进程管理是目前开发尤其是互联网web、服务器后端很常用的工具，尤其上云之后，云应用普遍推崇的microservices微服务架构进一步增加了后端进程数。而goreman很适合开发环境使用，能够一键式管理多个后台进程，并及时清理环境。不过真正的生产环境，还是使用monit/m、supervisor等更成熟稳定、功能全面的多进程管理工具。

