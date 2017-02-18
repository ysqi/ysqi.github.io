
---
date: 2017-02-18T10:33:41+08:00
title: "Go1_8graceful优雅的重启HTTP服务"
description: ""
disqus_identifier: 1487385221009631577
slug: "Go-1_8-http-graceful-ti-yan"
source: "https://segmentfault.com/a/1190000008383272"
tags: 
- golang 
topics:
- 编程语言与开发
---

很高兴Go 1.8发布了，这是个值得庆祝的日子。

如何优雅的关闭http服务在Go Web开发中一直被提及和讨论的话题，今天Go
1.8的发布终于为我们带来了这个特性。

文档中是这样介绍的：

    func (srv *Server) Shutdown(ctx context.Context) error

`Shutdown`
将无中断的关闭正在活跃的连接，然后平滑的停止服务。处理流程如下：

-   首先关闭所有的监听

-   然后关闭所有的空闲连接

-   然后无限期等待连接处理完毕转为空闲，并关闭

-   如果提供了 带有超时的`Context`，将在服务关闭前返回
    `Context`的超时错误

需要注意的是，`Shutdown` 并不尝试关闭或者等待 `hijacked`连接，如
`WebSockets`。如果需要的话调用者需要分别处理诸如长连接类型的等待和关闭。

其实，你只要调用 `Shutdown` 方法就好了。

简单示例：

    // main.go

    package main

    import (
        "fmt"
        "log"
        "net/http"
        "os"
        "os/signal"
        "syscall"
        "time"
    )

    func main() {
        http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
            fmt.Fprintf(w, "Hello World, %v\n", time.Now())
        })

        s := &http.Server{
            Addr:           ":8080",
            Handler:        http.DefaultServeMux,
            ReadTimeout:    10 * time.Second,
            WriteTimeout:   10 * time.Second,
            MaxHeaderBytes: 1 << 20,
        }

        go func() {
            log.Println(s.ListenAndServe())
            log.Println("server shutdown")
        }()

        // Handle SIGINT and SIGTERM.
        ch := make(chan os.Signal)
        signal.Notify(ch, syscall.SIGINT, syscall.SIGTERM)
        log.Println(<-ch)

        // Stop the service gracefully.
        log.Println(s.Shutdown(nil))

        // Wait gorotine print shutdown message
        time.Sleep(time.Second * 5)
        log.Println("done.")
    }

运行程序：

    go run main.go

然后 `ctrl + c` 终止该程序，会打印出如下信息：

    2017/02/17 11:36:28 interrupt
    2017/02/17 11:36:28 <nil>
    2017/02/17 11:36:28 http: Server closed
    2017/02/17 11:36:28 server shutdown

可以看到，服务被正确关闭了。

在没有
`Shutdown`方法之前，`ctrl + c`就硬生生的终止了，然后就没有然后了。

更多内容见官方文档：

[https://golang.org/pkg/net/ht...](https://golang.org/pkg/net/http/#Server.Shutdown)

