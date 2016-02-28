---
book_chapter: "0320"
book_chapter_name: "练习：map"
book_name: Golang入门指南
date: "2016-02-26T17:46:28.4470283+08:00"
description: ""
disqus_identifier: book000103020
slug: "exercise-map"
title: Golang入门指南-练习：map
codeurl: "https://wide.b3log.org/playground/.go"
---




实现 `WordCount`。它应当返回一个含有 `s` 中每个 “词” 个数的 map。函数 `wc.Test` 针对这个函数执行一个测试用例，并输出成功还是失败。

你会发现 [[https://go-zh.org/pkg/strings/#Fields][strings.Fields]] 很有帮助。

```
// +build OMIT

package main

import (
	"golang.org/x/tour/wc"
)

func WordCount(s string) map[string]int {
	return map[string]int{"x": 1}
}

func main() {
	wc.Test(WordCount)
}

```

