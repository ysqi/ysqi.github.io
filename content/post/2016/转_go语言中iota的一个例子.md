
---
date: 2016-12-31T11:34:31+08:00
title: "go语言中iota的一个例子"
description: ""
disqus_identifier: 1485833671733472204
slug: "goyu-yan-zhong-iotade-yi-ge-li-zi"
source: "https://segmentfault.com/a/1190000002717925"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

    package main
    import (
        "fmt"
    )
    type BitFlag int
    const (
        // iota为0，1左移0位 = 1
        Active BitFlag = 1 << iota
        // Send <=> Active <=> 1 << iota，此时iota为1，1左移1位 = 2
        Send
        // Receive <=> Send <=> 1 << iota，此时iota为2，1左移2位 = 4
        Receive
    )
    func main() {
        fmt.Println(Active, Send, Receive)
    }

iota是在编译的时候，编译器根据代码中iota与const关键字的位置动态替换的。

    package main

    import (
        "fmt"
    )

    const (
        //e=0,f=0,g=0
        e, f, g = iota, iota, iota
    )

    func main() {
        fmt.Println(e, f, g)
    }

可以将iota理解为const语句的行索引

    package main

    import (
        "fmt"
    )

    func main() {
        fmt.Println(iota)
    }

编译错误：undefined: iota.

------------------------------------------------------------------------

iota是预先声明的标识符，但是只能作用在const常量声明里。\
我怎么觉得iota这东西是go的私生子，只能被关在某个地方，不同于true/false等这些兄弟，不能访问它。

