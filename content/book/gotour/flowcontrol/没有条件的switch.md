---
book_chapter: "2.11"
book_chapter_name: "没有条件的switch"
book_name: Golang入门指南
date: "2016-02-26 17:45:30.5687179 +0800 CST"
description: ""
disqus_identifier: book000102011
slug: ""
title: Golang入门指南-没有条件的switch
codeurl: "https://wide.b3log.org/playground/.go"
---




没有条件的 switch 同 `switch`true` 一样。

这一构造使得可以用更清晰的形式来编写长的 if-then-else 链。

```
// +build OMIT

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

