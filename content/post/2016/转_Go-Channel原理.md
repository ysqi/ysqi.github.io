
---
date: 2016-12-31T11:34:51+08:00
title: "Go-Channel原理"
description: ""
disqus_identifier: 1485833691906397743
slug: "Go---Channel-yuan-li"
source: "https://segmentfault.com/a/1190000000507018"
tags: 
- channels 
- golang 
categories:
- 编程语言与开发
---

> 注：该文原文为 [Channel
> Axioms](http://dave.cheney.net/2014/03/19/channel-axioms) ，作者是
> Dave Cheney，这是他的[博客地址](http://dave.cheney.net/)

大部分的新的 Go 程序员能快速理解 channel 是作为一个 queue 的值和认同当
channel 是满的或者是空的时候， 操作是阻塞的概念。

这篇文章探讨了 channel 四个不太常见的特性：

-   给一个 nil channel 发送数据，造成永远阻塞
-   从一个 nil channel 接收数据，造成永远阻塞
-   给一个已经关闭的 channel 发送数据，引起 panic
-   从一个已经关闭的 channel 接收数据，立即返回一个零值

给一个 nil channel 发送数据，造成永远阻塞
-----------------------------------------

这第一个例子对于新来者是有点小惊奇的，它给一个 nil channel
发送数据，造成永远阻塞。

以下这个程序将在第5行造成死锁，因为未初始化的 channel 是 nil
的，其值是[零](http://golang.org/ref/spec#The_zero_value)

    package main

    func main() {
            var c chan string
            c <- "let's get started" // deadlock
    }

点击这里[运行](http://play.golang.org/p/1i4SjNSDWS)

从一个 nil channel 接收数据，造成永远阻塞
-----------------------------------------

类似的，从一个 nil channel 接收数据，会造成接受者永远阻塞。

    package main

    import "fmt"

    func main() {
            var c chan string
            fmt.Println(<-c) // deadlock
    }

点击这里[运行](http://play.golang.org/p/tjwSfLi7x0)

为什么会发生这样的情况？下面是一个可能的解释

-   channel 的 buffer 的大小不是类型声明的一部分，因此它必须是 channel
    的值的一部分
-   如果 channel 未被初始化，它的 buffer 的大小将是0
-   如果 channel 的 buffer 大小是0，那么它将没有 buffer
-   如果 channel 没有 buffer，一个发送将会被阻塞，直到另外一个 goroutine
    为接收做好了准备
-   如果 channel 是 nil
    的，并且接收者和发送者没有任何交互，他们都会阻塞然后在各自的 channel
    中等待以及不再被解除阻塞状态

给一个已经关闭的 channel 发送数据，引起 panic
---------------------------------------------

以下程序将有可能
panic，因为在它的兄弟姐妹有时间完成发送他们的值之前，这第一个 goroutine
在达到10的时候将关闭 channel。

    package main

    import "fmt"

    func main() {
            var c = make(chan int, 100)
            for i := 0; i < 10; i++ {
                    go func() {
                            for j := 0; j < 10; j++ {
                                    c <- j
                            }
                            close(c)
                    }()
            }
            for i := range c {
                    fmt.Println(i)
            }
    }

点击这里[运行](http://play.golang.org/p/hxUVqy7Qj-)

因此为什么没有一个 close() 版本能让你检测 channel 是否关闭？

    if !isClosed(c) {
            // c isn't closed, send the value
            c <- v
    }

但是这个函数有一个内在的竞争，某个人可能在我们检查完 isClosed(c)
之后，但是代码获取 c &lt;- v 之前关闭这个 channel。

处理这个问题的方法在被连接在该文章底部的 [2nd
article](http://blog.golang.org/pipelines) 被讨论。

从一个已经关闭的 channel 接收数据，立即返回一个零值
---------------------------------------------------

这最后一个示例与前一个是相反的，一旦一个 channel
被关闭，它的所有的值都会从 buffer 中流失，channel 将立即返回0值。

    package main

    import "fmt"

    func main() {
                c := make(chan int, 3)
                c <- 1
                c <- 2
                c <- 3
                close(c)
                for i := 0; i < 4; i++ {
                            fmt.Printf("%d ", <-c) // prints 1 2 3 0
                }
    }

点击这里[运行](http://play.golang.org/p/ajtVMsu8EO)

针对这个问题的正确的解决办法是使用 `range` 循环处理：

    for v := range c {
                // do something with v
    }

    for v, ok := <- c; ok ; v, ok = <- c {
                // do something with v
    }

这两个语句在函数中是相等的，展示 `range` 是做什么。

扩展阅读
--------

-   [Concurrency is not
    Parallelism](http://blog.golang.org/concurrency-is-not-parallelism)
    by Rob Pike
-   [Go Concurrency Patterns: Pipelines and
    cancellation](http://blog.golang.org/pipelines)
-   [Curious
    Channels](http://dave.cheney.net/2013/04/30/curious-channels)


