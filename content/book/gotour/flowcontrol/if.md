---
book_chapter: "0205"
book_chapter_name: "if"
book_name: Golang入门指南
date: "2016-02-26T17:45:28.2155833+08:00"
description: ""
disqus_identifier: book00010205
slug: "if"
title: Golang入门指南-if
codeurl: "https://wide.b3log.org/playground/e998f08ff23d425b9889bb4c2d78e602.go"
---

就像 `for` 循环一样，Go 的 `if` 语句也不要求用 `( )` 将条件括起来，同时， `{ }` 还是必须有的。

```Go
package main

import (
	"fmt"
	"math"
)

func sqrt(x float64) string {
	if x < 0 {
		return sqrt(-x) + "i"
	}
	return fmt.Sprint(math.Sqrt(x))
}

func main() {
	fmt.Println(sqrt(2), sqrt(-4))
}

```

