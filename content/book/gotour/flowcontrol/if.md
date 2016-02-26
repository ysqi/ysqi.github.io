---
book_chapter: "2.5"
book_chapter_name: "if"
book_name: Golang入门指南
date: "2016-02-26 17:45:28.2155833 +0800 CST"
description: ""
disqus_identifier: book00010205
slug: ""
title: Golang入门指南-if
codeurl: "https://wide.b3log.org/playground/.go"
---




就像 `for` 循环一样，Go 的 `if` 语句也不要求用 `(`)` 将条件括起来，同时， `{`}` 还是必须有的。

```
// +build OMIT

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

