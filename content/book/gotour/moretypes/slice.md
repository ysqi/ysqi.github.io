---
book_chapter: "0307"
book_chapter_name: "slice"
book_name: Golang入门指南
date: "2016-02-26T17:46:23.4027398+08:00"
description: ""
disqus_identifier: book00010307
slug: ""
title: Golang入门指南-slice
codeurl: "https://wide.b3log.org/playground/8dc0bb52d6e710727d4b2dfd5ab24bb0.go"
---

数组的长度不可改变，在特定场景中这样的集合就不太适用，Go中提供了一种灵活，功能强悍的内置类型Slices切片,与数组相比切片的长度是`不固定的`，可以追加元素，在追加时可能使切片的容量增大。

切片中有两个概念：一是len长度，二是cap容量，长度是指已经被赋过值的最大下标+1，可通过内置函数`len()`获得。容量是指切片目前可容纳的最多元素个数，可通过内置函数`cap()`获得。

切片是引用类型，因此在当传递切片时将引用同一指针，修改值将会影响其他的对象。

一个 slice 会指向一个序列的值，并且包含了长度信息。

`[]T` 是一个元素类型为 `T` 的 slice。 

<!-- ```go
package main

import "fmt"

func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	fmt.Println("s ==", s)

	for i := 0; i < len(s); i++ {
		fmt.Printf("s[%d] == %d\n", i, s[i])
	}
}

``` -->
