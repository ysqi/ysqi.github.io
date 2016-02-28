---
book_chapter: "0206"
book_chapter_name: "if的便捷语句"
book_name: Golang入门指南
date: "2016-02-26T17:45:28.6036055+08:00"
description: ""
disqus_identifier: book00010206
slug: "if-with-a-short-statement"
title: Golang入门指南-if的便捷语句
codeurl: "https://wide.b3log.org/playground/.go"
---




跟 `for` 一样， `if` 语句可以在条件之前执行一个简单语句。

由这个语句定义的变量的作用域仅在 `if` 范围之内。

（在最后的 `return` 语句处使用 `v` 看看。）

```
// +build OMIT

package main

import (
	"fmt"
	"math"
)

func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	}
	return lim
}

func main() {
	fmt.Println(
		pow(3, 2, 10),
		pow(3, 3, 20),
	)
}

```

