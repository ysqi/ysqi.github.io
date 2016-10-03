---
book_chapter: "0210"
book_chapter_name: "switch的执行顺序"
book_name: Golang入门指南
date: "2016-02-26T17:45:30.167695+08:00"
description: ""
disqus_identifier: book000102010
slug: "switch-evaluation-order"
title: Golang入门指南-switch的执行顺序
codeurl: "https://wide.b3log.org/playground/8ea40df1b2c22c5ee8524b8152f4499c.go"
---

switch 的条件从上到下的执行，当匹配成功的时候停止。

（例如，

	switch i {
		case 0:
		case f():
	}

当 `i==0` 时不会调用 `f()` ）

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("When's Saturday?")
	today := time.Now().Weekday()
	switch time.Saturday {
	case today + 0:
		fmt.Println("Today.")
	case today + 1:
		fmt.Println("Tomorrow.")
	case today + 2:
		fmt.Println("In two days.")
	default:
		fmt.Println("Too far away.")
	}
}

```

