---
book_chapter: "0413"
book_chapter_name: "Web服务器"
book_name: Golang入门指南
date: "2016-02-26T17:53:44.3809623+08:00"
description: ""
disqus_identifier: book000104013
slug: "http-web-server"
title: Golang入门指南-Web服务器
#codeurl: "https://wide.b3log.org/playground/.go"
---

[包 http](https://go-zh.org/pkg/net/http/#Handle) 通过任何实现了 `http.Handler` 的值来响应 HTTP 请求：

	package http

	type Handler interface {
		ServeHTTP(w ResponseWriter, r *Request)
	}

在这个例子中，类型 `Hello` 实现了 `http.Handler`。

访问 http://localhost:4000/ 会看到来自程序的问候。
	
**注意：** 这个例子无法在基于 web 的指南用户界面运行。为了尝试编写 
web 服务器，可能需要[安装 Go](https://go-zh.org/doc/install/)。

```Go
package main

import (
	"fmt"
	"log"
	"net/http"
)

type Hello struct{}

func (h Hello) ServeHTTP(
	w http.ResponseWriter,
	r *http.Request) {
	fmt.Fprint(w, "Hello!")
}

func main() {
	var h Hello
	err := http.ListenAndServe("localhost:4040", h)
	if err != nil {
		log.Fatal(err)
	}
}

```

