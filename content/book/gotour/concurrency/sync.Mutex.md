---
book_chapter: "0509"
book_chapter_name: "sync.Mutex"
book_name: Golang入门指南
date: "2016-02-26T17:42:53.9547601+08:00"
description: ""
disqus_identifier: book00010509
slug: ""
title: Golang入门指南-sync.Mutex
codeurl: "https://wide.b3log.org/playground/ef102da0a6051e26674d9073352a6ba6.go"
---

我们已经看到 `channel` 用来在各个 goroutine 间进行通信是非常合适的了。

但是如果我们并不需要通信呢？比如说，如果我们只是想保证在每个时刻，只有一个 goroutine
能访问一个共享的变量从而避免冲突？

这里涉及的概念叫做 **互斥**，通常使用 _互斥锁_(mutex)_来提供这个限制。

Go 标准库中提供了 [`sync.Mutex`](https://go-zh.org/pkg/sync/#Mutex) 类型及其两个方法：

- `Lock`
- `Unlock`

我们可以通过在代码前调用 `Lock` 方法，在代码后调用 `Unlock` 方法来保证一段代码的互斥执行。
参见 `Inc` 方法。

我们也可以用 `defer` 语句来保证互斥锁一定会被解锁。参见 `Value` 方法。

```Go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter 的并发使用是安全的。
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc 增加给定 key 的计数器的值。
func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	// Lock 之后同一时刻只有一个 goroutine 能访问 c.v
	c.v[key]++
	c.mux.Unlock()
}

// Value 返回给定 key 的计数器的当前值。
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	// Lock 之后同一时刻只有一个 goroutine 能访问 c.v
	defer c.mux.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}

```

