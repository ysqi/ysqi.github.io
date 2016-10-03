---
book_chapter: "0414"
book_chapter_name: "练习：HTTP处理"
book_name: Golang入门指南
date: "2016-02-26T17:53:44.8719904+08:00"
description: "exercise-http"
disqus_identifier: book000104014
slug: ""
title: Golang入门指南-练习：HTTP处理
#codeurl: "https://wide.b3log.org/playground/.go"
---

实现下面的类型，并在其上定义 ServeHTTP 方法。在 web 服务器中注册它们来处理指定的路径。

	type String string

	type Struct struct {
		Greeting string
		Punct    string
		Who      string
	}
例如，可以使用如下方式注册处理方法：

	http.Handle("/string", String("I'm a frayed knot."))
	http.Handle("/struct", &Struct{"Hello", ":", "Gophers!"})

在启动你的 http 服务器后，你将能够访问：
http://localhost:4000/string 和 http://localhost:4000/struct.

**注意：** 这个例子无法在基于 web 的指南用户界面运行。为了尝试编写 
web 服务器，可能需要[安装 Go](https://go-zh.org/doc/install/)。

```go
package main

import (
	"log"
	"net/http"
)
type String string

type Struct struct {
    Greeting string
    Punct    string
    Who      string
}

func main() {
	http.Handle("/string", String("I'm a frayed knot."))
	http.Handle("/struct", &Struct{"Hello", ":", "Gophers!"})

	log.Fatal(http.ListenAndServe("localhost:40", nil))
}
```

