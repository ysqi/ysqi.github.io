---
book_chapter: "2.7"
book_chapter_name: "if和else"
book_name: Golang入门指南
date: "2016-02-26 17:45:28.997628 +0800 CST"
description: ""
disqus_identifier: book00010207
slug: ""
title: Golang入门指南-if和else
codeurl: "https://wide.b3log.org/playground/.go"
---




在 `if` 的便捷语句定义的变量同样可以在任何对应的 `else` 块中使用。

（提示：两个 `pow` 调用都在 `main` 调用 `fmt.Println` 前执行完毕了。）

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
	} else {
		fmt.Printf("%g >= %g\n", v, lim)
	}
	// 这里开始就不能使用 v 了
	return lim
}

func main() {
	fmt.Println(
		pow(3, 2, 10),
		pow(3, 3, 20),
	)
}

```

