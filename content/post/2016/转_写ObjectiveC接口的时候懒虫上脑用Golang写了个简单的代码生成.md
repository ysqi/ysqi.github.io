
---
date: 2016-12-31T11:34:47+08:00
title: "写ObjectiveC接口的时候懒虫上脑用Golang写了个简单的代码生成"
description: ""
disqus_identifier: 1485833687703650877
slug: "xie-ObjectiveCjie-kou-de-shi-hou-lan-chong-shang-nao-yong--Golang-xie-le-ge-jian-chan-de-dai-ma-she"
source: "https://segmentfault.com/a/1190000000646284"
tags: 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

    package main

    import "fmt"

    func main() {
        fName := "clientupdateVersion"
        pName := "version"
        otherParameters := [] string {}
        printFuncHeader(fName, pName)
        printFuncParameter(otherParameters)
    }

    func printFuncHeader(fName string, pName string) {
        fmt.Printf("+ (NSDictionary *)%s:(NSString *)%s", fName, pName)
        return 
    }

    func printFuncParameter(otherParameters [] string) {
        for _, value := range otherParameters {
            fmt.Printf("\n%s:(NSString *)%s", value, value)
        }
        fmt.Println(";")
        return
    }

> 好吧, 的确很low, 有空研究下如何自动写入到系统剪贴板,
> 这样就可以偷懒不去复制了...

