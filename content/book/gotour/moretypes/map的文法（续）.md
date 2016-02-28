---
book_chapter: "0318"
book_chapter_name: "map的文法（续）"
book_name: Golang入门指南
date: "2016-02-26T17:46:27.688985+08:00"
description: ""
disqus_identifier: book000103018
slug: "map-literals-con"
title: Golang入门指南-map的文法（续）
codeurl: "https://wide.b3log.org/playground/.go"
---




如果顶级的类型只有类型名的话，可以在文法的元素中省略键名。

```
// +build OMIT

package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google":    {37.42202, -122.08408},
}

func main() {
	fmt.Println(m)
}

```

