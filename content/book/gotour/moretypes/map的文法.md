---
book_chapter: "0317"
book_chapter_name: "map的文法"
book_name: Golang入门指南
date: "2016-02-26T17:46:27.3469654+08:00"
description: ""
disqus_identifier: book000103017
slug: "map-literals"
title: Golang入门指南-map的文法
codeurl: "https://wide.b3log.org/playground/866bd55dd8ec30d05aa290ffadad0444.go"
---

map 的文法跟[结构体]({{< ref "结构体.md">}})文法相似，不过必须有键名。

```Go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}

func main() {
	fmt.Println(m)
}

```

