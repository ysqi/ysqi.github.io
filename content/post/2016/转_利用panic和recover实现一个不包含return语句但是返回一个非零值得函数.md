
---
date: 2016-12-31T11:32:53+08:00
title: "利用panic和recover实现一个不包含return语句但是返回一个非零值得函数"
description: ""
disqus_identifier: 1485833573383311995
slug: "li-yong-panic-he--recover-shi-xian-yi-ge-bu-bao-han-return-yu-gou-dan-shi-fan-hui-yi-ge-fei-ling-zhi"
source: "https://segmentfault.com/a/1190000007468641"
tags: 
- golang 
categories:
- 编程语言与开发
---

如题，这个问题是The Go Programming Language
里面的练习5.19，挺有意思的一道题目。\
中文版可以参考
[https://shifei.me/gopl-zh/ch5...](https://shifei.me/gopl-zh/ch5/ch5-10.html)

实现代码如下：

    package main

    import "fmt"

    func main() {
        a := returnN()
        fmt.Println(a)
    }

    func returnN() (result int) {
        defer func() {
            if p := recover(); p != nil {
                result = p.(int)
            }
        }()
        panic(3)
    }

运行后，屏幕会打印出3。原因我认为是以下两点：

1.  导致panic异常的函数不会继续运行，但能正常返回。

2.  被延迟执行的匿名函数可以修改函数返回给调用者的返回值。

所以在panic 函数执行后，函数准备返回result 这个变量，之后执行defer
的func，在这个func 里改变了result 的值，从而实现了问题的要求。

