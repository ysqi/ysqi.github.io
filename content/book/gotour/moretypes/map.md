---
book_chapter: "3.16"
book_chapter_name: "map"
book_name: Golang入门指南
date: "2016-02-26 17:46:26.9809445 +0800 CST"
description: ""
disqus_identifier: book000103016
slug: ""
title: Golang入门指南-map
codeurl: "https://wide.b3log.org/playground/.go"
---




map 映射键到值。

map 在使用之前必须用 `make` 来创建；值为 `nil` 的 map 是空的，并且不能对其赋值。

```
// +build OMIT

package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex

func main() {
	m = make(map[string]Vertex)
	m["Bell Labs"] = Vertex{
		40.68433, -74.39967,
	}
	fmt.Println(m["Bell Labs"])
}

```

