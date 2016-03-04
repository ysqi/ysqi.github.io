---
book_chapter: "0505"
book_chapter_name: "select"
book_name: Golang入门指南
date: "2016-02-26T17:42:52.7626919+08:00"
description: ""
disqus_identifier: book00010505
slug: ""
title: Golang入门指南-select
codeurl: "https://wide.b3log.org/playground/84b0bcb4362dad7cd68f9ad221c62ef4.go"
---

`select` 语句使得一个 goroutine 在多个通讯操作上等待。

`select` 会阻塞，直到条件分支中的某个可以继续执行，这时就会执行那个条件分支。当多个都准备好的时候，会**随机**选择一个。

```go
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}

```

