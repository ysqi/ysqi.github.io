
---
date: 2017-02-18T11:13:49+08:00
title: "go协程与主线程强占运行"
description: ""
disqus_identifier: 1487387629128548171
slug: "go-xie-cheng-yu-zhu-xian-cheng-jiang-zhan-yun-hang"
source: "http://blog.csdn.net/qo2yycc2/article/details/55251241"
tags: 
- golang 
categories:
- 编程语言与开发
---

最近在学习了go 语言 ,  正好学习到了 协程这一块
,遇到了困惑的地方.这个是go语言官方文档 .
在我的理解当中是,协程只能在主线程释放时间片后才会经过系统调度来运行协程,其实正确的也确实是这样的,但是我遇到了协程强占主线程的一个问题,经过帮助,现在已经了解.废话不多说,先看代码

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    go say("world")
    say("hello")
    /*
        fmt.Println("---------------1")

        a := []int{7, 2, 8, -9, 4, 0}
        fmt.Println("===", a[:len(a)/2])
        c := make(chan int)
        go sum(a[:len(a)/2], c)
        go sum(a[len(a)/2:], c)
        x, y := <-c, <-c // receive from c

        fmt.Println(x, y, x+y)

        fmt.Println("---------------2")

        c2 := make(chan int, 2)
        c2 <- 1
        c2 <- 2
        fmt.Println(<-c2)
        fmt.Println(<-c2)

        fmt.Println("---------------3")
        c3 := make(chan int, 10)
        go fibonacci(cap(c3), c3)
        for i := range c3 {
            fmt.Println(i)
        }

        fmt.Println("---------------4")
        c4 := make(chan int)
        quit := make(chan int)
        go func() {
            for i := 0; i < 10; i++ {
                fmt.Println(<-c4)
            }
            quit <- 0
        }()
        fibonacci2(c4, quit)

        fmt.Println("---------------5")
        tick := time.Tick(100 * time.Millisecond)
        boom := time.After(500 * time.Millisecond)
        for {
            select {
            case <-tick:
                fmt.Println("tick. ")
            case <-boom:
                fmt.Println("BOOM!")
                return
            default:
                fmt.Println("    .")
                time.Sleep(50 * time.Millisecond)
            }
        }*/
}

func say(s string) {
    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
}
```

先看两次代码运行结果

第一次:  (结合上面代码查看打印顺序)

![](2017021813/998499-20170215163930988-764196126.png)

 

 第二次:(结合第一次查看打印顺序)

![](2017021814/998499-20170215164048113-1267931342.png)

 

是不是发现了每次的打印顺序是不同的

这个就是协程强占执行\

我们先来看下它的执行循序 ,
主线程运行====\>释放时间片====\>协程运行==\>释放时间片====\>主线程运行

根据这段代码 
```
say("hello")
```
我们知道,这个是属于主线程里面的,所以优先执行(注意实参是"hello")  

然后看看 say 方法 
```go
func say(s string) {
    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
}
```
当执行到循环里面的 
```go
    time.Sleep(100 * time.Millisecond)
```
会释放时间片,同时 暂停执行代码,系统调度到协程 
```go
    go say("world")
```
 

也是同一个方法,同时也会执行
```go
    time.Sleep(100 * time.Millisecond)
```
释放时间片 

于是再打印的时候就会出现强占执行

 

