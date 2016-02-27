---
book_chapter: "3.13"
book_chapter_name: "range"
book_name: Golang入门指南
date: "2016-02-26T17:46:25.9118833+08:00"
description: ""
disqus_identifier: book000103013
slug: ""
title: Golang入门指南-range
codeurl: "https://wide.b3log.org/playground/.go"
---




`for` 循环的 `range` 格式可以对 slice 或者 map 进行迭代循环。

当使用 `for` 循环遍历一个 slice 时，每次迭代 `range` 将返回两个值。
第一个是当前下标（序号），第二个是该下标所对应元素的一个拷贝。

```
// +build OMIT

package main

import "fmt"

var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

func main() {
	for i, v := range pow {
		fmt.Printf("2**%d = %d\n", i, v)
	}
}

```

