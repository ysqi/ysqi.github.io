
---
date: 2016-12-31T11:34:55+08:00
title: "GoByExample系列:非阻塞Channels操作"
description: ""
disqus_identifier: 1485833695554181131
slug: "Go-By-Example-ji-lie-:fei-zu-sai--Channels-cao-zuo"
source: "https://segmentfault.com/a/1190000000492788"
tags: 
- select 
- non-blocking-channel 
- channels 
- golang 
topics:
- 编程语言与开发
---

> 注：该系列文章全部来自 [Go By Example](https://gobyexample.com/)
> 系列翻译而来，个人翻译水平以及理解水平有限，如要更加精确的理解，请看原文[Go
> by Example: Non-Blocking Channel
> Operations](https://gobyexample.com/non-blocking-channel-operations)。

在 channels （信道？） 上基本的 sends （发送） 和 receives
（接收）是阻塞模式的。尽管如此， 我们可以使用 select 和一个 default
子句来非阻塞的 sends、receives，甚至是非阻塞的多路选择。

> 注：感谢@lidashuang的说明提醒，文章没有描述清楚，修改如下：select默认是阻塞的，但在select里面还有default语法，这类似于switch，default就是当监听的channel都没有准备好的时候，默认执行的（select不再阻塞等待channel）\
> 同时，有时候会出现goroutine阻塞的情况，可以利用select设置超时来避免整个程序进入阻塞状态

代码版本一
----------

代码如下：

    package main
    import "fmt"
    func main() {
        messages := make(chan string)
        signals := make(chan bool) 

    /**
    这里是一个非阻塞 receive。如果在 messages 上的值是可用的，那 select 将 <-messages 的值带上，执行 <-messages 下面的println语句。如果不是，它将立即带上 default 的值，执行 default 下面的println语句
    **/
        select {
        case msg := <-messages:
            fmt.Println("received message", msg)
        default:
            fmt.Println("no message received")
        }
    /**
    一个非阻塞 send 的类似工作 
    **/
        msg := "hi"
        select {
        case messages <- msg:
            fmt.Println("sent message", msg)
        default:
            fmt.Println("no message sent")
        }

    /**
    我们可以用在 default 之上使用多个 cases 来实现一个非阻塞的多路 select。在这里我们尝试在 messages 和 signals 上实现非阻塞 receives。
    **/

        select {
        case msg := <-messages:
            fmt.Println("received message", msg)
        case sig := <-signals:
            fmt.Println("received signal", sig)
        default:
            fmt.Println("no activity")
        }
    }

最后程序的结果输出为：

    $ go run non-blocking-channel-operations.go 
    no message received
    no message sent
    no activity

代码版本二
----------

代码如下：

    package main

    import (
        "fmt"
    )

    func main() {
        // messages := make(chan string)    //如果不加缓存的话，就全部会选择defalut
        messages := make(chan string, 1) //加了缓存的话，会选择对应的
        signals := make(chan bool)

        // messages <- "test"

        select {
        case msg := <-messages:
            fmt.Println("received message", msg) //因为messages目前本身还没有值，因此选择default执行
        default:
            fmt.Println("no message received")
        }

        // go func() {
        msg := "hi world"
        // }()
        select {
        case messages <- msg:
            fmt.Println("sent message", msg) //因为channels有缓存，所以这里的msg发送到 channels messages 能处理，不会被阻塞住
        default:
            fmt.Println("no message sent")
        }

        select {
        case msg := <-messages:
            fmt.Println("received message", msg) //因为messages已经有值了，所以会选择这个case执行
        case sig := <-signals:
            fmt.Println("received signal", sig)
        default:
            fmt.Println("no activity")
        }
    }

输出结果如下：

    $ go run non-blocking-channel-operations.go 
    no message received
    sent message hi world
    received message hi world   

代码版本三
----------

代码如下：

    package main

    import (
        "fmt"
    )

    func main() {
        // messages := make(chan string)    //如果不加缓存的话，就全部会选择defalut
        messages := make(chan string, 1) //加了缓存的话，会选择对应的
        signals := make(chan bool)

        messages <- "test"

        select {
        case msg := <-messages:
            fmt.Println("received message", msg) //messages已经有值，因此选择这个case执行
        default:
            fmt.Println("no message received")
        }

        // go func() {
        msg := "hi world"
        // }()
        select {
        case messages <- msg:
            fmt.Println("sent message", msg)
        default:
            fmt.Println("no message sent")
        }

        select {
        case msg := <-messages:
            fmt.Println("received message", msg)
        case sig := <-signals:
            fmt.Println("received signal", sig)
        default:
            fmt.Println("no activity")
        }
    }

代码输出结果如下：

    $ go run non-blocking-channel-operations.go 
    received message test
    sent message hi world
    received message hi world

代码版本四
----------

代码如下：

    package main

    import (
        "fmt"
    )

    func main() {
        messages := make(chan string) //如果不加缓存的话，就全部会选择defalut
        //messages := make(chan string, 1) //加了缓存的话，会选择对应的
        signals := make(chan bool)

        messages <- "test" //因为该channels没有缓存，而其又赋值了，会到时死锁。

        select {
        case msg := <-messages:
            fmt.Println("received message", msg)
        default:
            fmt.Println("no message received")
        }

        // go func() {
        msg := "hi world"
        // }()
        select {
        case messages <- msg:
            fmt.Println("sent message", msg)
        default:
            fmt.Println("no message sent")
        }

        select {
        case msg := <-messages:
            fmt.Println("received message", msg)
        case sig := <-signals:
            fmt.Println("received signal", sig)
        default:
            fmt.Println("no activity")
        }
    }

代码输出结果为：

    fatal error: all goroutines are asleep - deadlock!

runtime goroutine
-----------------

runtime包中有几个处理goroutine的函数

1.  Goexit - 退出当前执行的goroutine，但是defer函数还会继续调用
2.  Gosched -
    让出当前goroutine的执行权限，调度器安排其它等待的任务运行，并在下次某个时候从该位置恢复执行
3.  NumCPU - 返回CPU核数量
4.  NumGoroutine - 返回正在执行和排队的任务总数
5.  GOMAXPROCS - 用来设置可以运行的CPU核数


