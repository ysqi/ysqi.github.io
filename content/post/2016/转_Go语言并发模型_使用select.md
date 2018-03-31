
---
date: 2016-12-31T11:33:07+08:00
title: "Go语言并发模型:使用select"
description: ""
disqus_identifier: 1485833587867829079
slug: "Goyu-yan-bing-fa-mo-xing-:shi-yong--select"
source: "https://segmentfault.com/a/1190000006815341"
tags: 
- select 
- concurrency 
- golang 
categories:
- 编程语言与开发
---

简介
----

作为一种现代语言，go语言实现了对并发的原生支持。上几期文章中，我们对goroutine
和 channel进行了详细的讲解。但是要实现对 channel
的控制，从语言层面上来说，select 语句是必不可少的部分。本文中，我们就
select 语句的行为和使用方法进行深入讨论。

阅读建议
--------

本文中的内容是
Go语言并发模型的一篇，但是与上几期关系不是特别密切，可以独立阅读。本文的内容源自于
[go language specifications]() 和 Rob Pike
在2012年进行的一场名为["concurrency"](https://talks.golang.org/2012/concurrency.slide#1)
的演讲。如果有时间的话，建议在 YouTube 上看一下他本人的演讲。

select 语句的行为
-----------------

为了便于理解，我们首先给出一个代码片段：

    // https://talks.golang.org/2012/concurrency.slide#32
    select {
    case v1 := <-c1:
        fmt.Printf("received %v from c1\n", v1)
    case v2 := <-c2:
        fmt.Printf("received %v from c2\n", v1)
    case c3 <- 23:
        fmt.Printf("sent %v to c3\n", 23)
    default:
        fmt.Printf("no one was ready to communicate\n")
    }

上面这段代码中，select 语句有四个 case 子语句，前两个是 receive
操作，第三个是 send 操作，最后一个是默认操作。代码执行到 select 时，case
语句会按照源代码的顺序被评估，且只评估一次，评估的结果会出现下面这几种情况：

1.  除 default 外，如果只有一个 case
    语句评估通过，那么就执行这个case里的语句；

2.  除 default 外，如果有多个 case
    语句评估通过，那么通过伪随机的方式随机选一个；

3.  如果 default 外的 case 语句都没有通过评估，那么执行 default
    里的语句；

4.  如果没有 default，那么 代码块会被阻塞，指导有一个 case
    通过评估；否则一直阻塞

如果 case 语句中 的 receive 操作的对象是 nil
channel，那么也会阻塞，下面我们看一个更全面、用法也更高级的例子：

    // https://golang.org/ref/spec#Select_statements
    var a []int
    var c, c1, c2, c3, c4 chan int
    var i1, i2 int
    select {
    case i1 = <-c1:
        print("received ", i1, " from c1\n")
    case c2 <- i2:
        print("sent ", i2, " to c2\n")
    case i3, ok := (<-c3):  // same as: i3, ok := <-c3
        if ok {
            print("received ", i3, " from c3\n")
        } else {
            print("c3 is closed\n")
        }
    case a[f()] = <-c4:
        // same as:
        // case t := <-c4
        //    a[f()] = t
    default:
        print("no communication\n")
    }

    for {  // 向 channel c 发送随机 bit 串
        select {
        case c <- 0:  // note: no statement, no fallthrough, no folding of cases
        case c <- 1:
        }
    }

    select {}  // 永久阻塞

注意：与 C/C++ 等传统编程语言不同，go语言的 case 语句不需要 break
关键字去跳出 select。

select 的使用
-------------

### 为请求设置超时时间

在 golang 1.7 之前， http 包并没有引入 context 支持，通过 http.Client
向一个坏掉的服务发送请求会导致响应缓慢。类似的场景下，我们可以使用
select 控制服务响应时间，下面是一个简单的demo：

    func main() {
        c := boring("Joe")
        timeout := time.After(5 * time.Second)
        for {
            select {
            case s := <-c:
                fmt.Println(s)
            case <-timeout:
                fmt.Println("You talk too much.")
                return
            }
        }
    }

### done channel

上几期的文章中，我们均讨论过 done
channel，它可以用于保证流水线上每个阶段goroutine 的退出。在
golang.org/x/net 包中，done channel 被广泛应用。这里我们看一眼
golang.org/x/net/context/ctxhttp 中 Do 方法的实现：

    // https://github.com/golang/net/blob/release-branch.go1.7/context/ctxhttp/ctxhttp.go

    // Do sends an HTTP request with the provided http.Client and returns
    // an HTTP response.
    //
    // If the client is nil, http.DefaultClient is used.
    //
    // The provided ctx must be non-nil. If it is canceled or times out,
    // ctx.Err() will be returned.
    func Do(ctx context.Context, client *http.Client, req *http.Request) (*http.Response, error) {
        if client == nil {
            client = http.DefaultClient
        }
        resp, err := client.Do(req.WithContext(ctx))
        // If we got an error, and the context has been canceled,
        // the context's error is probably more useful.
        if err != nil {
            select {
            case <-ctx.Done():
                err = ctx.Err()
            default:
            }
        }
        return resp, err
    } 

quit channel
============

在很多场景下，quit channel 和 done channel
是一个概念。在并发程序中，通常 main routine 将任务分给其它 go routine
去完成，而自身只是起到调度作用。这种情况下，main 函数无法知道
其它goroutine 任务是否完成，此时我们需要 quit
channel；对于更细粒度的控制，比如完成多少，还是需要 done channel
(参考WaitGroup)。 下面是 quit channel 的一个例子，首先是 main routine：

    // 创建 quit channel
    quit := make(chan string)
    // 启动生产者 goroutine
    c := boring("Joe", quit)
    // 从生产者 channel 读取结果
    for i := rand.Intn(10); i >= 0; i-- { fmt.Println(<-c) }
    // 通过 quit channel 通知生产者停止生产
    quit <- "Bye!"
    fmt.Printf("Joe says: %q\n", <-quit)

我们再看 生产者 go routine 中与 quit channel 相关的部分：

    select {
    case c <- fmt.Sprintf("%s: %d", msg, i):
        // do nothing
    case <-quit:
        cleanup()
        quit <- "See you!"
        return
    }

### Google Search (延伸阅读)

Google Search 是一个很经典的例子，由于代码较多，有兴趣的童鞋查看 [Rob
Pike 的 ppt](https://talks.golang.org/2012/concurrency.slide#42)。\
更高阶的并发方式可以阅读 Sameer Ajmani 的 ppt [Advanced Go Concurrency
Patterns](https://talks.golang.org/2013/advconc.slide)

并发相关的主题就先到这里，下一期文章中，我们会讨论go语言测试工具链中的单元测试。

### 相关链接：

1.  [Rob
    Pike演讲：concurrency](https://talks.golang.org/2012/concurrency.slide#31)

2.  [language specification: select
    statement](https://golang.org/ref/spec#Select_statements)

扫码关注微信公众号“深入Go语言”



