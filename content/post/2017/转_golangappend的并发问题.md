
---
date: 2017-02-18T11:13:44+08:00
title: "golangappend的并发问题"
description: ""
disqus_identifier: 1487387624870943945
slug: "golang-appendde-bing-fa-wen-ti"
source: "https://my.oschina.net/u/222608/blog/839602"
tags: 
- golang 
categories:
- 编程语言与开发
---

先看一段代码

    ackage main

    import (
        "fmt"
        "sync"
    )

    func main() {
        var wg sync.WaitGroup
        s := make([]int, 0, 1000)

        for i := 0; i < 1000; i++ {
            v := i
            wg.Add(1)
            go func() {
                s = append(s, v)
                wg.Done()
            }()
        }

        wg.Wait()
        fmt.Printf("%v\n", len(s))
    }

结果

    第一次：928
    第二次：945
    第三次：986
    ……

多运行几次你就会发现，slice长度并不是1000，而是不停的在变，为什么呢？

因为append并不是并发安全的。

我们举一个简单例子，比如，当A和B两个协程运行append的时候同时发现s[1]这个位置是空的，他们就都会把自己的值放在这个位置，这样他们两个的值就会覆盖，造成数据丢失。

那该怎么写？最简单的方式就是用锁，贴一个例子。

    package main

    import (
        "fmt"
        "sync"
    )

    func main() {
        var (
            wg    sync.WaitGroup
            mutex sync.Mutex
        )

        s := make([]int, 0, 1000)

        for i := 0; i < 1000; i++ {
            v := i
            wg.Add(1)
            go func() {
                mutex.Lock()
                s = append(s, v)
                mutex.Unlock()
                wg.Done()
            }()
        }

        wg.Wait()
        fmt.Printf("%v\n", len(s))
    }

运行一下这个例子就会发现，s的长度总是1000。

