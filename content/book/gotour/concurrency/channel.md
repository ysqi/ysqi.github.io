---
book_chapter: "0502"
book_chapter_name: "channel"
book_name: Golang入门指南
date: "2016-02-26T17:42:51.6056257+08:00"
description: ""
disqus_identifier: book00010502
slug: "channel"
title: Golang入门指南-channel
codeurl: "https://wide.b3log.org/playground/2f63a33271371ed83eb58dc922569c7b.go"
---

channel 是有类型的管道，可以用 channel 操作符 `<-` 对其发送或者接收值。

	ch <- v    // 将 v 送入 channel ch。
	v := <-ch  // 从 ch 接收，并且赋值给 v。

（“箭头”就是数据流的方向。）

和 map 与 slice 一样，channel 使用前必须创建：

	ch := make(chan int)

默认情况下，在另一端准备好之前，发送和接收都会阻塞。这使得 goroutine 可以在没有明确的锁或竞态变量的情况下进行同步。

```Go
package main

import "fmt"

func sum(a []int, c chan int) {
	sum := 0
	for _, v := range a {
		sum += v
	}
	c <- sum // 将和送入 c
}

func main() {
	a := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(a[:len(a)/2], c)
	go sum(a[len(a)/2:], c)
	x, y := <-c, <-c // 从 c 中获取

	fmt.Println(x, y, x+y)
}

```

