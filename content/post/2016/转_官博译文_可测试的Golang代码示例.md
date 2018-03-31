
---
date: 2016-12-31T11:33:30+08:00
title: "官博译文:可测试的Golang代码示例"
description: ""
disqus_identifier: 1485833610306978455
slug: "guan-bo-yi-wen-:ke-ce-shi-de--Golang-dai-ma-shi-li"
source: "https://segmentfault.com/a/1190000005608114"
tags: 
- 测试驱动开发 
- testing 
- golang 
categories:
- 编程语言与开发
---

简介
----

Dodoc 的 [示例](http://golang.org/pkg/testing/#hdr-Examples)
是一些可执行的测试代码的聚合，他们做为包的文档中的一部分提供给读者阅读和执行。读者可以点击
"Run" 按钮来测试代码。

Golang 的标准包包括很多这种代码示例（比如
[strings](http://golang.org/pkg/strings/#Contains) 包）

本文将示例如何写出类似的代码示例。

示例即单元测试
--------------

代码示例作为包的一部分编译并执行。

在典型的单元测试中，示例就是包内 \_test.go
文件中的一些方法。代码示例跟测试代码不同，示例方法以 Example
开头（不同于测试代码的 Test）并且没有参数。

    stringutil 包是 golang 代码示例仓库中的一部分。下面的代码展示了他是如何演示 Reverse 的用法。

    package stringutil_test

    import (
    "fmt"

    "github.com/golang/example/stringutil"
    )

    func ExampleReverse() {
    fmt.Println(stringutil.Reverse("hello"))
    // Output: olleh
    }

上述代码可以在 stringutil 目录下的 example\_test.go 文件中找到。

Godoc 在 Reverse 方法的文档中展示了这个代码示例，详见图：

