---
book_chapter: "0308"
book_chapter_name: "slice的slice"
book_name: Golang入门指南
date: "2016-02-26T17:46:23.7687608+08:00"
description: ""
disqus_identifier: book00010308
slug: "slices-of-slice"
title: Golang入门指南-slice的slice
codeurl: "https://wide.b3log.org/playground/71a72cd5a31afe7b3d466cb2740467be.go"
---

slice 可以包含任意的类型。 即你可以把 slice 当做是一组元素集合，集合的大小可变。

示例中是一个 slice 的元素也是一个 slice。

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	// Create a tic-tac-toe board.
	game := [][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	}

	// The players take turns.
	game[0][0] = "X"
	game[2][2] = "O"
	game[2][0] = "X"
	game[1][0] = "O"
	game[0][2] = "X"

	printBoard(game)
}

func printBoard(s [][]string) {
	for i := 0; i < len(s); i++ {
		fmt.Printf("%s\n", strings.Join(s[i], " "))
	}
}

```

