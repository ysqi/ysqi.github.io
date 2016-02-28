---
book_chapter: "0311"
book_chapter_name: "nilslice"
book_name: Golang入门指南
date: "2016-02-26T17:46:25.1968424+08:00"
description: ""
disqus_identifier: book000103011
slug: ""
title: Golang入门指南-nilslice
codeurl: "https://wide.b3log.org/playground/.go"
---




slice 的零值是 `nil` 。

一个 nil 的 slice 的长度和容量是 0。

```
// +build OMIT

package main

import "fmt"

func main() {
	var z []int
	fmt.Println(z, len(z), cap(z))
	if z == nil {
		fmt.Println("nil!")
	}
}

```

