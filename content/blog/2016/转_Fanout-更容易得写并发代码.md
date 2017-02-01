
---
date: 2016-12-31T11:34:37+08:00
title: "Fanout-更容易得写并发代码"
description: ""
disqus_identifier: 1485833677774162909
slug: "Fanout---geng-rong-yi-de-xie-bing-fa-dai-ma"
source: "https://segmentfault.com/a/1190000002486136"
tags: 
- 并发 
- golang 
topics:
- 编程语言与开发
---

不用管理 grouting 和 channel 了。 以下为转发 golangtc.com 原文：

------------------------------------------------------------------------

刚刚写了个包，觉得值得出来分享下：

文档: <https://github.com/sunfmin/fanout>

用来简化并发程序(goroutine, channel)的编写，这个包直接改写自Go
Concurrency Patterns: Pipelines博客的最后一个样例程序。

平时写带goroutine和channel的程序，总是时不时的爆"fatal error: all
goroutines are asleep - deadlock!"，检查起来又很难找原因。

例子程序 - 用60个goroutine一起跑whois来查看域名是不是还在:

    inputs := []interface{}{}


    for _, word:= range domainWords {
        inputs = append(inputs, word)
    }

    results, err2 := fanout.ParallelRun(60, func(input interface{})
    (interface{}, error) {
        word := input.(string)
        if strings.TrimSpace(word) == "" {
            return nil, nil
        }

        py := pinyin.Convert(word)
        pydowncase := strings.ToLower(py)
        domain := pydowncase + ".com"
        outr, err := domainAvailable(word, domain)

        if outr.available {
            fmt.Printf("[Ohh Yeah] %s %s\n", outr.word, outr.domain)
        } else {
            fmt.Printf("\t\t\t %s %s %s\n", outr.word, outr.domain, outr.summary)
        }

        if err != nil {
            fmt.Println("Error: ", err)
        }

        return outr, nil
    }, inputs)

    fmt.Println("Finished ", len(results), ", Error:", err2)

一图来说明:



