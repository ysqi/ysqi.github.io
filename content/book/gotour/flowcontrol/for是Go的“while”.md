---
book_chapter: "2.3"
book_chapter_name: "for是Go的“while”"
book_name: Golang入门指南
date: "2016-02-26 17:45:27.45854 +0800 CST"
description: ""
disqus_identifier: book00010203
slug: ""
title: Golang入门指南-for是Go的“while”
codeurl: "https://wide.b3log.org/playground/.go"
---




基于此可以省略分号：C 的 `while` 在 Go 中叫做 `for` 。

```
// +build OMIT

package main

import "fmt"

func main() {
	sum := 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}

```

