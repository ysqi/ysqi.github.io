---
book_chapter: "0406"
book_chapter_name: "Stringers"
book_name: Golang入门指南
date: "2016-02-26T17:53:40.6737503+08:00"
description: ""
disqus_identifier: book00010406
slug: ""
title: Golang入门指南-Stringers
codeurl: "https://wide.b3log.org/playground/.go"
---




一个普遍存在的接口是 [[https://go-zh.org/pkg/fmt/][`fmt`]] 包中定义的 [[https://go-zh.org/pkg/fmt/#Stringer][`Stringer`]]。

	type Stringer interface {
		String() string
	}

`Stringer` 是一个可以用字符串描述自己的类型。`fmt`包
（还有许多其他包）使用这个来进行输出。

```
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

func main() {
	a := Person{"Arthur Dent", 42}
	z := Person{"Zaphod Beeblebrox", 9001}
	fmt.Println(a, z)
}

```

