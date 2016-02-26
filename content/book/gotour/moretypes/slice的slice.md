---
book_chapter: "3.8"
book_chapter_name: "slice的slice"
book_name: Golang入门指南
date: "2016-02-26 17:46:23.7687608 +0800 CST"
description: ""
disqus_identifier: book00010308
slug: ""
title: Golang入门指南-slice的slice
codeurl: "https://wide.b3log.org/playground/.go"
---




slice 可以包含任意的类型，包括另一个 slice。

```
// +build OMIT

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

