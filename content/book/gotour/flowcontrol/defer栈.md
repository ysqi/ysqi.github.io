---
book_chapter: "0213"
book_chapter_name: "defer栈"
book_name: Golang入门指南
date: "2016-02-26T17:45:31.339762+08:00"
description: ""
disqus_identifier: book000102013
slug: "defer-multi"
title: Golang入门指南-defer栈
codeurl: "https://wide.b3log.org/playground/6900e8a91ee83c1fe0e10a9fd1f02329.go"
---

延迟的函数调用被压入一个栈中。当函数返回时，
会按照后进先出的顺序调用被延迟的函数调用。

阅读[博文]( http://blog.go-zh.org/defer-panic-and-recover)了解更多关于 `defer` 语句的信息。

```go
package main

import "fmt"

func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}

```

