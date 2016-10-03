---
book_chapter: "0504"
book_chapter_name: "range和close"
book_name: Golang入门指南
date: "2016-02-26T17:42:52.3466681+08:00"
description: ""
disqus_identifier: book00010504
slug: "channel-close"
title: Golang入门指南-关闭channel
codeurl: "https://wide.b3log.org/playground/cd8448053f2dafbc766386a4c9d011ff.go"
---

发送者可以 `close` 一个 channel 来表示再没有值会被发送了。

接收者可以通过赋值语句的第二参数(bool)来判断 channel 是否被关闭：当没有值可以接收并且 channel 已经被关闭，那么经过

	v, ok := <-ch

之后 `ok` 会被设置为 `false`。

循环:

	for i := range c
会不断从 channel 接收值，直到它被关闭。

**注意：** 只有发送者才能关闭 channel，而不是接收者。向一个已经关闭的 channel 发送数据会引起 panic。

**还要注意：** channel 与文件不同；通常情况下无需关闭它们。只有在需要告诉接收者没有更多的数据的时候才有必要进行关闭，例如中断一个 `range`。

```go
package main

import (
	"fmt"
)

func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}

```

