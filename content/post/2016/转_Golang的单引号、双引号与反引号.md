
---
date: 2016-12-31T11:33:52+08:00
title: "Golang的单引号、双引号与反引号"
description: ""
disqus_identifier: 1485833632818755881
slug: "Golangde-chan-yin-hao-、shuang-yin-hao-yu-fan-yin-hao"
source: "https://segmentfault.com/a/1190000004850183"
tags: 
- golang 
categories:
- 编程语言与开发
---

Go语言的字符串类型`string`在本质上就与其他语言的字符串类型不同：

-   Java的String、C++的std::string以及Python3的str类型都只是定宽字符序列

-   Go语言的字符串是一个用UTF-8编码的变宽字符序列，它的每一个字符都用一个或多个字节表示

即：**一个Go语言字符串是一个任意字节的常量序列**。

Golang的双引号和反引号都可用于表示一个常量字符串，不同在于：

-   双引号用来创建可解析的字符串字面量(支持转义，但不能用来引用多行)

-   反引号用来创建原生的字符串字面量，这些字符串可能由多行组成(不支持任何转义序列)，原生的字符串字面量多用于书写多行消息、HTML以及正则表达式

而单引号则用于表示Golang的一个特殊类型：`rune`，类似其他语言的`byte`但又不完全一样，是指：**码点字面量（Unicode
code point）**，不做任何转义的原始内容。

------------------------------------------------------------------------

参考链接：[https://crazyof.me/blog/archives/2539.ht...](https://crazyof.me/blog/archives/2539.html)

