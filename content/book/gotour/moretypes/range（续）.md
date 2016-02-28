---
book_chapter: "0314"
book_chapter_name: "range（续）"
book_name: Golang入门指南
date: "2016-02-26T17:46:26.2689038+08:00"
description: ""
disqus_identifier: book000103014
slug: "range-continued"
title: Golang入门指南-range（续）
codeurl: "https://wide.b3log.org/playground/.go"
---




可以通过赋值给 `_` 来忽略序号和值。

如果只需要索引值，去掉 “ `,`value` ” 的部分即可。

```
// +build OMIT

package main

import "fmt"

func main() {
	pow := make([]int, 10)
	for i := range pow {
		pow[i] = 1 << uint(i)
	}
	for _, value := range pow {
		fmt.Printf("%d\n", value)
	}
}

```

