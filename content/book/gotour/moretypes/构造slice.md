---
book_chapter: "3.10"
book_chapter_name: "构造slice"
book_name: Golang入门指南
date: "2016-02-26 17:46:24.8428222 +0800 CST"
description: ""
disqus_identifier: book000103010
slug: ""
title: Golang入门指南-构造slice
codeurl: "https://wide.b3log.org/playground/.go"
---




slice 由函数 `make` 创建。这会分配一个全是零值的数组并且返回一个 slice 指向这个数组：

	a := make([]int, 5)  // len(a)=5

为了指定容量，可传递第三个参数到 `make`：

	b := make([]int, 0, 5) // len(b)=0, cap(b)=5

	b = b[:cap(b)] // len(b)=5, cap(b)=5
	b = b[1:]      // len(b)=4, cap(b)=4

```
// +build OMIT

package main

import "fmt"

func main() {
	a := make([]int, 5)
	printSlice("a", a)
	b := make([]int, 0, 5)
	printSlice("b", b)
	c := b[:2]
	printSlice("c", c)
	d := c[2:5]
	printSlice("d", d)
}

func printSlice(s string, x []int) {
	fmt.Printf("%s len=%d cap=%d %v\n",
		s, len(x), cap(x), x)
}

```

