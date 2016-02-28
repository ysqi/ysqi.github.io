---
book_chapter: "0501"
book_chapter_name: "goroutine"
book_name: Golang入门指南
date: "2016-02-26T17:42:50.4965623+08:00"
description: ""
disqus_identifier: book00010501
slug: ""
title: Golang入门指南-goroutine
codeurl: "https://wide.b3log.org/playground/.go"
---




_goroutine_ 是由 Go 运行时环境管理的轻量级线程。

	go f(x, y, z)

开启一个新的 goroutine 执行

	f(x, y, z)

`f`，`x`，`y` 和 `z` 是当前 goroutine 中定义的，但是在新的 goroutine 中运行 `f`。

goroutine 在相同的地址空间中运行，因此访问共享内存必须进行同步。[[https://go-zh.org/pkg/sync/][`sync`]] 提供了这种可能，不过在 Go 中并不经常用到，因为有其他的办法。（在接下来的内容中会涉及到。）

```
// +build OMIT

package main

import (
	"fmt"
	"time"
)

func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("world")
	say("hello")
}

```

