---
book_chapter: "5.3"
book_chapter_name: "缓冲channel"
book_name: Golang入门指南
date: "2016-02-26T17:42:51.9786471+08:00"
description: ""
disqus_identifier: book00010503
slug: ""
title: Golang入门指南-缓冲channel
codeurl: "https://wide.b3log.org/playground/.go"
---




channel 可以是 _带缓冲的_。为 `make` 提供第二个参数作为缓冲长度来初始化一个缓冲 channel：

	ch := make(chan int, 100)

向带缓冲的 channel 发送数据的时候，只有在缓冲区满的时候才会阻塞。
而当缓冲区为空的时候接收操作会阻塞。

修改例子使得缓冲区被填满，然后看看会发生什么。

```
// +build OMIT

package main

import "fmt"

func main() {
	ch := make(chan int, 2)
	ch <- 1
	ch <- 2
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}

```

