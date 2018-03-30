
---
date: 2016-12-31T11:34:02+08:00
title: "Golang时间格式化的奇怪设定-为什么你一直出错"
description: ""
disqus_identifier: 1485833642435411802
slug: "Golang-shi-jian-ge-shi-hua-de-ji-guai-she-ding----wei-shen-me-ni-yi-zhi-chu-cuo"
source: "https://segmentfault.com/a/1190000004222341"
tags: 
- go语言 
- golang 
topics:
- 编程语言与开发
---

首发于：<http://blog.shajiquan.com/2015/12/golang-time-format/>

今天有人在群里问：

    问一个时间格式化问题：
    fmt.Println(time.Now().Format("2006year 01month 02day"))
    2015year 12month 18day

    fmt.Println(time.Now().Format("2015year 01month 01day"))
    181253year 12month 12day
    为什么？
    困扰好久

首先，这是一个很奇葩的问题。

其次，我，以及比我对 Golang
更了解的朋友，都掉过这个坑。我们曾在这个问题上，花了很多时间，最后发现是
Golang 自己的奇怪设定导致。尤其是，一段时间不用 time
包后，过段时间居然又忘了。

然后，怪我们没看文档，可是...

结论：年、月、日、时、分、秒，英文、数字，必须精确地限定到 golang
指定的时间原点：`2006-01-02 15:04:05`

示例：

    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        fmt.Println("hello ")

        now := time.Now()

        nowRight := now.Format("2006-01-02 15:04")
        fmt.Println(nowRight)

        nowWrong := now.Format("2006-01-03日错了 15:05 分错了")
        fmt.Println(nowWrong)

        fmt.Println(now.Format("Mon Jan _2 15:04:05 2006 年"))
        fmt.Println("变态吧！")
    }

附，格式化字符串模板：

    const (
        ANSIC       = "Mon Jan _2 15:04:05 2006"
        UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
        RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
        RFC822      = "02 Jan 06 15:04 MST"
        RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
        RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
        RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
        RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
        RFC3339     = "2006-01-02T15:04:05Z07:00"
        RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
        Kitchen     = "3:04PM"
        // Handy time stamps.
        Stamp      = "Jan _2 15:04:05"
        StampMilli = "Jan _2 15:04:05.000"
        StampMicro = "Jan _2 15:04:05.000000"
        StampNano  = "Jan _2 15:04:05.000000000"
    )

