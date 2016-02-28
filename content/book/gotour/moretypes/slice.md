---
book_chapter: "0307"
book_chapter_name: "slice"
book_name: Golang入门指南
date: "2016-02-26T17:46:23.4027398+08:00"
description: ""
disqus_identifier: book00010307
slug: ""
title: Golang入门指南-slice
codeurl: "https://wide.b3log.org/playground/.go"
---




一个 slice 会指向一个序列的值，并且包含了长度信息。

`[]T` 是一个元素类型为 `T` 的 slice。

`len(s)` 返回 slice s 的长度。

```
// +build OMIT

package main

import "fmt"

func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	fmt.Println("s ==", s)

	for i := 0; i < len(s); i++ {
		fmt.Printf("s[%d] == %d\n", i, s[i])
	}
}

```

