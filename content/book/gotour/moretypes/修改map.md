---
book_chapter: "0319"
book_chapter_name: "修改map"
book_name: Golang入门指南
date: "2016-02-26T17:46:28.057006+08:00"
description: ""
disqus_identifier: book000103019
slug: "mutating-maps"
title: Golang入门指南-修改map
codeurl: "https://wide.b3log.org/playground/.go"
---




在 map `m` 中插入或修改一个元素：

	m[key] = elem

获得元素：

	elem = m[key]

删除元素：

	delete(m, key)

通过双赋值检测某个键存在：

	elem, ok = m[key]

如果 `key` 在 `m` 中， `ok` 为 `true`。否则， `ok` 为 `false`，并且 `elem` 是 map 的元素类型的零值。

同样的，当从 map 中读取某个不存在的键时，结果是 map 的元素类型的零值。

```
// +build OMIT

package main

import "fmt"

func main() {
	m := make(map[string]int)

	m["Answer"] = 42
	fmt.Println("The value:", m["Answer"])

	m["Answer"] = 48
	fmt.Println("The value:", m["Answer"])

	delete(m, "Answer")
	fmt.Println("The value:", m["Answer"])

	v, ok := m["Answer"]
	fmt.Println("The value:", v, "Present?", ok)
}

```

