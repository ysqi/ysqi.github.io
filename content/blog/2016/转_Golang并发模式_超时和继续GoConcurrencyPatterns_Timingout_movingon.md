
---
date: 2016-12-31T11:18:42+08:00
title: "Golang 并发模式超时和继续 Go Concurrency Patterns: Timing out moving on"
description: 
disqus_identifier: 1485832722802940286
slug: Golang-bing-fa-mo-shi-chao-shi-he-ji-xu--Go-Concurrency-Patterns-Timing-out-moving-on
source: https://segmentfault.com/a/1190000005616829
tags: 
- goroutine 
- 并发 
- golang 
topics:
- 编程语言与开发
---

翻译自 Go Blog。\
原文地址：<https://blog.golang.org/go-concurrency-patterns-timing-out-and>

并发编程有自己的一些习惯用语，超时就是其中之一。虽然 Golang
的管道并没有直接支持超时，但是实现起来并不难。假设遇到了这样一种场景：在从
管道 ch 中取值之前至少等待 1
秒钟。我们可以创建一个管道用来传递信号，开启一个协程休眠一秒钟，然后给管道传递一个值。

    timeout := make(chan bool, 1)
    go func() {
        time.Sleep(1 * time.Second)
        timeout <- true
    }()

然后就可以使用一个 select 语句来从 timeout 或者 ch 管道中获取数据。如果
ch 管道在 1 秒钟之后还没有返回数据，超时的判断条件就会触发 ch
的读操作将会被抛弃掉。

    select {
        case <-ch:
        // a read from ch has occurred
        case <-timeout:
        // the read from ch has timed out
    }

timeout 管道的缓冲区空间为 1，因此 timeout
协程将会在发送消息到管道之后退出执行。协程并不知道（也不关心）管道中的值是否被接受。因此，即使
ch 管道先于 timeout 管道返回了，timeout 协程也不会永久等待。timeout
管道最终会被垃圾回收机制回收掉。

（在上面的示例中我们使用了 time.Sleep
方法来演示协程和管道的机制，但是在真实的代码中应该用 time.After
方法，该方法返回了一个管道，并且会在参数指定的时间之后向管道中写入一个消息）

下面我们来看这种模式的另外一个变种。我们需要从多个分片数据库中同时取数据，程序只需要其中最先返回的那个数据。

下面的 Query
方法接受两个参数：一个数据库链接的切片和一个数据库查询语句。该方法将平行查询所有数据库并返回第一个接受到的响应结果。

    func Query(conns []Conn, query string) Result {
        ch := make(chan Result, 1)
        for _, conn := range conns {
            go func(c Conn) {
                select {
                case ch <- c.DoQuery(query):
                default:
                }
            }(conn)
        }
        return <-ch
    }

在上面这个例子中，go 关键字后的闭包实现了一个非阻塞式的查询请求，因为
DoQuery 方法被放到了带 default 分支的 select 语句中。假如 DoQuery
方法没有立即返回，default 分支将会被选中执行。让查询请求非阻塞保证了 for
循环中创建的协程不会一直阻塞。另外，假如在主方法从 ch
管道中取出值并返回结果之前有第二个查询结果返回了，管道 ch
的写操作将会失败，因为管道并未就绪。

上面描述的问题其实是“竞争关系”的一种典型例子，示例代码只是一种很通俗的解决方案。我们只是中管道上设定了缓冲区（通过在管道的
make
方法中传入第二个参数），保证了第一个写入者能够有空间来写入值。这种策略保证了管道的第一次写入一定会成功，无论代码以何种顺序执行，第一个写入的值将会被当作最终的返回值。

Go 的协程之前可以进行复杂的协作，以上两个例子就是最简单的证明。

