---
book_chapter: "0411"
book_chapter_name: "练习：Reader"
book_name: Golang入门指南
date: "2016-02-26T17:53:43.5089125+08:00"
description: "exercise-reader"
disqus_identifier: book000104011
slug: ""
title: Golang入门指南-练习：Reader
codeurl: "https://wide.b3log.org/playground"
---

实现一个 `Reader` 类型，它不断生成 ASCII 字符 `'A'` 的流。

```Go
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

// TODO: Add a Read([]byte) (int, error) method to MyReader.

func main() {
	reader.Validate(MyReader{})
}

```

