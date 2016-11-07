---
book_chapter: "0212"
book_chapter_name: "defer"
book_name: Golang入门指南
date: "2016-02-26T17:45:30.9537399+08:00"
description: ""
disqus_identifier: book000102012
slug: ""
title: Golang入门指南-defer
codeurl: "https://wide.b3log.org/playground/7b74145ffe6487beb3bf4a5068cbc93b.go"
---

defer 语句会延迟函数的执行直到上层函数返回。

延迟调用的参数会立刻生成，但是在上层函数返回前函数都不会被调用。

defer 执行顺序是先进后出。

```Go
package main

import "fmt"

func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}

```

