
---
date: 2016-12-31T11:34:31+08:00
title: "go语言中import不允许循环包含"
description: ""
disqus_identifier: 1485833671227252921
slug: "goyu-yan-zhong-importbu-yun-hu-xun-huan-bao-han"
source: "https://segmentfault.com/a/1190000002726107"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

go的包不允许循环包含，具体例子：

main.go:

    package main

    import (
        "fmt"
        "test/pkg1"
    )

    func main() {
        fmt.Println("in main.main")
        fmt.Printf("pkg1.Black=%s\n", pkg1.Black)
        fmt.Printf("pkg2.Black=%s\n", pkg2.Black)
    }

    func init() {
        fmt.Println("in main.init")
        fmt.Printf("pkg1.Black=%s\n", pkg1.Black)
        fmt.Printf("pkg2.Black=%s\n", pkg2.Black)
    }

pkg1.go:

    package pkg1

    import (
        "fmt"
        "test/pkg2"
    )

    const (
        Black string = "#000"
        white string = "#fff"
    )

    func init() {
        fmt.Println("in pkg1.init")
        fmt.Printf("pkg2.Black=%s\n", pkg2.Black)
    }

pkg2.go:

    package pkg2

    import (
        "fmt"
        "test/pkg1"
    )

    const (
        Black string = "#000"
        white string = "#fff"
    )

    func init() {
        fmt.Println("in pkg2.init")
        fmt.Printf("pkg1.Black=%s\n", pkg1.Black)
    }

go build报错:

    import cycle not allowed

