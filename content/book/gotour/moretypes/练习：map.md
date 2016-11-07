---
book_chapter: "0320"
book_chapter_name: "练习：map"
book_name: Golang入门指南
date: "2016-02-26T17:46:28.4470283+08:00"
description: ""
disqus_identifier: book000103020
slug: "exercise-map"
title: Golang入门指南-练习：map
codeurl: "https://wide.b3log.org/playground/f5974d93879f7dd44635d34c9317d8c3.go"
---
实现 `WordCount`。它应当返回一个含有 `s` 中每个 “词” 个数的 map。 

你会发现[strings.Fields](https://go-zh.org/pkg/strings/#Fields) 很有帮助。

```Go
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

