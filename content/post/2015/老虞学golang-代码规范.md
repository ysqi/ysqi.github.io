---
date: 2013-04-02
title: 老虞学Golang-代码规范
slug: ysqi-golang-code-formate
topics:
- 编程语言与开发
tags:
- Golang
- 笔记
- 教程
books:
- 老虞学GoLang
disqus_identifier: 100011
---

开始一项新语言前需要先了解该语言的语法(如果你有其他语言的编程知识的话)，开始学习前，我们一起了解下Go的格式。
如果大家都统一编码风格，那么在维护他人代码时就能带来便利。同时我们在提交代码前执行一次fmt命令，以便提交统一风格的代码。

<!--more-->

### 注释
Go支持C语言风格的“//”块注释，也支持C++风格的行注释，同时可使用/**/进行包的 注释. 我们看string包的源代码，使用//注释了包，方法以及行。我们需要养成好的习惯，尽量去多写些注释，这样不但有利于自己以后的回顾，已给他人阅读你的代码提供了方便，当然Go下的源代码使用Go命令能够生成文档，而文档的描述内容源自注释，在编码阶段就同步书写注释，而不要在整理代码时书写注释(此时的思维没有编码时清晰，补救中总容易丢失些东西)。

```Go
// Copyright 2009 The Go Authors. All rights reserved.
// Use of this source code is Governed by a BSD-style
// license that can be found in the LICENSE file.

// Package strings implements simple functions to manipulate strings.
  package strings

  import (
          "unicode"
          "utf8"
  )

// explode splits s into an array of UTF-8 sequences, one per Unicode character (still strings) up to a maximum of n (n < 0 means no limit).
// Invalid UTF-8 sequences become correct encodings of U+FFF8.
  func explode(s string, n int) []string {
          if n == 0 {
                  return nil
          }
          l := utf8.RuneCountInString(s)
          if n <= 0 || n > l {
                  n = l
          }
          a := make([]string, n)
          var size, rune int
          i, cur := 0, 0
          for ; i+1 < n; i++ {
                  rune, size = utf8.DecodeRuneInString(s[cur:])
                  a[i] = string(rune)
                  cur += size
          }
          // add the rest, if there is any
          if cur < len(s) {
                  a[i] = s[cur:]
          }
          return a
  }
```

### 命名

在Go中名称不但具有表达含义的功能，同时也具有约束使用的特点。如果一个函数的名称是小写的则表示该函数不能在其他包中使用。

命名必须使用骆驼命名法，而不能使用下划线法。

任何需要对外暴露的名字必须大写字母开头，不需要暴露在包外的名字必须以小写字母开头。

接口的命令，按照惯例，如果接口只有一个方法，则该接口命名为方法名成加上”ER“后缀。

### 分号

Go和C语言一样使用`;`来结束一个语句，但不一样的是，在Go中由编译器去处理`;`，所以你必须在编写代码是省略`;`。当然也有例外，for循环(使用;将初始部分、条件部分和遍历元素区分)，一行中有多个语句，多赋值语句等。

需要注意，不能将控制语句(for,if else,switch,select)的左括号另起一行。如
```Go
//错误的方式
if(1==2)
{
fmt.Println("God!")
}


//正确的书写
if 1 == 2  {
    fmt.Println("God!")
}
```

学到新的知识后，再补充。

 ---------2013-04-10 补充---------------------

  Go语言编程规范：

      官方地址：http://golang.org/doc/effective_go.html    （墙了）

      中文翻译：http://zh-golang.appsp0t.com/ref/spec?lang=en

------------------end------------------------------
