---
book_chapter: "0309"
book_chapter_name: "对slice切片"
book_name: Golang入门指南
date: "2016-02-26T17:46:24.4638005+08:00"
description: ""
disqus_identifier: book00010309
slug: "slices-to-slices"
title: Golang入门指南-对slice切片
codeurl: "https://wide.b3log.org/playground/5006e1af96fd58ac27f1a1266869806d.go"
---

slice 可以重新切片，创建一个新的 slice 值指向相同的数组。

表达式

	s[lo:hi]

表示从 `lo` 到 `hi-1` 的 slice 元素。一个左闭右开区间`[lo,hi)`。

因此 `s[lo:lo]`是空的，而`s[lo:lo+1]`有一个元素。


1. 重建全部元素的 slice
```go
s2 := s[:]
```

2. 缺省hi时, hi=len(s)
```go
s2 := s[lo:]
```

3. 缺省lo时,lo=0
```go	
s2 := s[:hi]
```
 

<!-- ```go

package main

import "fmt"

func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	fmt.Println("s ==", s)
	fmt.Println("s[1:4] ==", s[1:4])

	// 省略下标代表从 0 开始
	fmt.Println("s[:3] ==", s[:3])

	// 省略上标代表到 len(s) 结束
	fmt.Println("s[4:] ==", s[4:])
}

``` -->

