---
book_chapter: "0211"
book_chapter_name: "没有条件的switch"
book_name: Golang入门指南
date: "2016-02-26T17:45:30.5687179+08:00"
description: ""
disqus_identifier: book000102011
slug: "switch-with-no-condition"
title: Golang入门指南-没有条件的switch
codeurl: "https://wide.b3log.org/playground/b7cb36325458480b83b94f12aec2d68e.go"
---


没有条件的 switch 同 `switch true` 一样。

	switch true	{
		//case ...
	}

这一构造使得可以用更清晰的形式来编写长的 if-then-else 链。

```Go
package main

import (
	"fmt"
	"time"
)

func main() {
	t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
}

```

