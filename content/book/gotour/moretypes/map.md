---
book_chapter: "0316"
book_chapter_name: "map"
book_name: Golang入门指南
date: "2016-02-26T17:46:26.9809445+08:00"
description: ""
disqus_identifier: book000103016
slug: "map"
title: Golang入门指南-map
codeurl: "https://wide.b3log.org/playground/3eae4b105abfb940f1ce5e2b87de060f.go"
---

map 映射键到值。

map 在使用之前必须用 `make` 来创建；值为 `nil` 的 map 是空的，并且不能对其赋值。

```Go
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

