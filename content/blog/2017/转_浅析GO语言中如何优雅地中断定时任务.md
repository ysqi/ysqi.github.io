
---
date: 2017-02-18T11:13:38+08:00
title: "浅析GO语言中如何优雅地中断定时任务"
description: ""
disqus_identifier: 1487387618626054919
slug: "jian-xi-GOyu-yan-zhong-ru-he-you-ya-de-zhong-duan-ding-shi-ren-wu"
source: "https://yq.aliyun.com/articles/69303?utm_campaign=wenzhang&amp;utm_medium=article&amp;utm_source=QQ-qun&amp;utm_content=m_10072"
tags: 
- golang 
topics:
- 编程语言与开发
---

### 问题描述

现在我们创建了一个定时器，能定时的去做某件事，并且在执行时间超时的时候，能把这个定时器关掉。例如需要收集一周的日志，创建一个定时任务去收集日志，每5秒钟执行一次，一周的时间过后需要停掉这个定时任务。

### 标准库Ticker

标准库提供里的Ticker类，主要功能是定时重复的去做某件事情，如果没有设定超时，它会一直执行下去。常见的写法如下：

    t := time.NewTicker(3 * time.Second)
    timeout := time.After(10 * time.Second)
    go func() {
            for {   
                    <-t.C
                     ...
            }       
    }()
    <-timeout
    ...

`注意到这个Ticker对象是无法关闭的`，好的，你可能会发现Ticker类提供了Stop方法。但是我们看看如果你这样去关闭t的话，会出现什么情况。

    package main

    import (
            "fmt"
            "time"
    )

    func DoTickerWork(res chan interface{}, timeout <-chan time.Time) {
            t := time.NewTicker(3 * time.Second)
            go func() {
                    defer close(res)
                    i := 1
                    for {
                            <-t.C
                            fmt.Printf("start %d th worker\n", i)
                            res <- i
                            i++
                    }
            }()
            <-timeout
            t.Stop()
            return
    }

    func main() {
            res := make(chan interface{}, 10000)
            timeout := time.After(10 * time.Second)
            DoTickerWork(res, timeout)
            for v := range res {
                    fmt.Println(v)
            }
    }

直觉上来看，新起的goroutine在等待的过程中，主线程会把定时器关掉，似乎没有什么bug，然而输出是这样：

    $go run ticker.go 
    start 1 th worker
    start 2 th worker
    start 3 th worker
    1
    2
    3
    fatal error: all goroutines are asleep - deadlock!

    goroutine 1 [chan receive]:
    main.main()
        /home/gepin.zs/go/src/timer/ticker.go:29 +0xad

    goroutine 6 [chan receive]:
    main.DoTickerWork.func1(0xc42006c060, 0xc4200161c0)
        /home/gepin.zs/go/src/timer/ticker.go:14 +0x8e
    created by main.DoTickerWork
        /home/gepin.zs/go/src/timer/ticker.go:19 +0x60
    exit status 2

这说明Ticker对象的stop方法`并没有关掉`这个Ticker的channel，而只是`阻止了channel的数据写入`，所以goroutine的任务依然在进行中，但是\<-t.C一直阻塞，出现了deadlock的情况。可能会有人说调用close(t.C)就可以了，但是编译会报错：cannot
close receive-only channel， 因为t.C是一个只读队列，无法调用close方法。

### 怎么解决

不要以为stop就可以关掉Ticker了，我们可以新建一个名字为done的channel，缓存大小为1，goroutine里面采用select，然后尝试获取timeout，如果能够取到，说明已经触发超时，然后close(done)，这个时候任务结束，主线程return。代码如下：

    package main

    import (
            "fmt"
            "time"
    )

    func DoTickerWork(res chan interface{}, timeout <-chan time.Time) {
            t := time.NewTicker(3 * time.Second)
            done := make(chan bool, 1)
            go func() {
                    defer close(res)
                    i := 1
                    for {
                            select {
                            case <-t.C:
                                    fmt.Printf("start %d th worker\n", i)
                                    res <- i
                                    i++
                            case <-timeout:
                                    close(done)
                                    return
                            }
                    }
            }()
            <-done
            return
    }

    func main() {
            res := make(chan interface{}, 10000)
            timeout := time.After(10 * time.Second)
            DoTickerWork(res, timeout)
            for v := range res {
                    fmt.Println(v)
            }
    }

程序返回结果

    $go run ticker.go
    start 1 th worker
    start 2 th worker
    start 3 th worker
    1
    2
    3

