---
book_chapter: "2.10"
book_chapter_name: "switch的执行顺序"
book_name: Golang入门指南
date: "2016-02-26 17:45:30.167695 +0800 CST"
description: ""
disqus_identifier: book000102010
slug: ""
title: Golang入门指南-switch的执行顺序
codeurl: "https://wide.b3log.org/playground/.go"
---




switch 的条件从上到下的执行，当匹配成功的时候停止。

（例如，

	switch i {
	case 0:
	case f():
	}

当 `i==0` 时不会调用 `f` 。）

#appengine: 注意：Go playground 中的时间总是从 2009-11-10 23:00:00 UTC 开始，
#appengine: 如何校验这个值作为一个练习留给读者完成。

```
// +build OMIT

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

