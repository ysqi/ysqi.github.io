---
book_chapter: "0202"
book_chapter_name: "for（续）"
book_name: Golang入门指南
date: "2016-02-26T17:45:27.0705178+08:00"
description: ""
disqus_identifier: book00010202
slug: "for-continued"
title: Golang入门指南-for（续）
codeurl: "https://wide.b3log.org/playground/a6b0b9c42075661a4e8713f48585daa2.go"
---


循环初始化语句和后置语句都是可选的。

```Go
package main

import "fmt"

func main() {
	sum := 1
	for ; sum < 1000; {
		sum += sum
	}
	fmt.Println(sum)
}

```

