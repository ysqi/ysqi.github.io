
---
date: 2017-02-18T11:13:49+08:00
title: "go协程与主线程强占运行"
description: ""
disqus_identifier: 1487387629128548171
slug: "go-xie-cheng-yu-zhu-xian-cheng-jiang-zhan-yun-hang"
source: "http://blog.csdn.net/qo2yycc2/article/details/55251241"
tags: 
- golang 
topics:
- 编程语言与开发
---

最近在学习了go 语言 ,  正好学习到了 协程这一块
,遇到了困惑的地方.这个是go语言官方文档 .
在我的理解当中是,协程只能在主线程释放时间片后才会经过系统调度来运行协程,其实正确的也确实是这样的,但是我遇到了协程强占主线程的一个问题,经过帮助,现在已经了解.废话不多说,先看代码

 

     1 package main
     2 
     3 import (
     4     "fmt"
     5     "time"
     6 )
     7 
     8 func main() {
     9     go say("world")
    10     say("hello")
    11     /*
    12         fmt.Println("---------------1")
    13 
    14         a := []int{7, 2, 8, -9, 4, 0}
    15         fmt.Println("===", a[:len(a)/2])
    16         c := make(chan int)
    17         go sum(a[:len(a)/2], c)
    18         go sum(a[len(a)/2:], c)
    19         x, y := <-c, <-c // receive from c
    20 
    21         fmt.Println(x, y, x+y)
    22 
    23         fmt.Println("---------------2")
    24 
    25         c2 := make(chan int, 2)
    26         c2 <- 1
    27         c2 <- 2
    28         fmt.Println(<-c2)
    29         fmt.Println(<-c2)
    30 
    31         fmt.Println("---------------3")
    32         c3 := make(chan int, 10)
    33         go fibonacci(cap(c3), c3)
    34         for i := range c3 {
    35             fmt.Println(i)
    36         }
    37 
    38         fmt.Println("---------------4")
    39         c4 := make(chan int)
    40         quit := make(chan int)
    41         go func() {
    42             for i := 0; i < 10; i++ {
    43                 fmt.Println(<-c4)
    44             }
    45             quit <- 0
    46         }()
    47         fibonacci2(c4, quit)
    48 
    49         fmt.Println("---------------5")
    50         tick := time.Tick(100 * time.Millisecond)
    51         boom := time.After(500 * time.Millisecond)
    52         for {
    53             select {
    54             case <-tick:
    55                 fmt.Println("tick. ")
    56             case <-boom:
    57                 fmt.Println("BOOM!")
    58                 return
    59             default:
    60                 fmt.Println("    .")
    61                 time.Sleep(50 * time.Millisecond)
    62             }
    63         }*/
    64 }
    65 
    66 func say(s string) {
    67     for i := 0; i < 5; i++ {
    68         time.Sleep(100 * time.Millisecond)
    69         fmt.Println(s)
    70     }
    71 }

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

    1 say("hello")

我们知道,这个是属于主线程里面的,所以优先执行(注意实参是"hello")  

然后看看 say 方法 

    1 func say(s string) {
    2     for i := 0; i < 5; i++ {
    3         time.Sleep(100 * time.Millisecond)
    4         fmt.Println(s)
    5     }
    6 }

当执行到循环里面的 

    time.Sleep(100 * time.Millisecond)

会释放时间片,同时 暂停执行代码,系统调度到协程 

    go say("world")

 

也是同一个方法,同时也会执行

    time.Sleep(100 * time.Millisecond)

释放时间片 

于是再打印的时候就会出现强占执行

 

