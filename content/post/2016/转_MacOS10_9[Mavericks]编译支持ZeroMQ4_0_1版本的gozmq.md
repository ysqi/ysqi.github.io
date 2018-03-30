
---
date: 2016-12-31T11:35:12+08:00
title: "MacOS10_9[Mavericks]编译支持ZeroMQ4_0_1版本的gozmq"
description: ""
disqus_identifier: 1485833712859187677
slug: "Mac-OS-10_9-[Mavericks]-bian-yi-zhi-chi--ZeroMQ-4_0_1-ban-ben-de--gozmq"
source: "https://segmentfault.com/a/1190000000343907"
tags: 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

下载 4.0.1 版本的 ZeroMQ 代码后解压到任意目录。

    tar -xzvf zeromq-4.0.1.tar.gz
    cd zeromq-4.0.1
    ./configure --prefix=/usr
    make
    make install
    go get -tags zmq_4_x github.com/alecthomas/gozmq

测试程序

    package main

    import (
      "fmt"
      zmq "github.com/alecthomas/gozmq"
      "os"
    )

    func main() {
      major, minor, patch := zmq.Version()
      fmt.Printf("Current 0MQ version is %d.%d.%d\n", major, minor, patch)
      os.Exit(0)
    }

运行测试程序

    $ go run version.go
    $ Current 0MQ version is 4.0.1

