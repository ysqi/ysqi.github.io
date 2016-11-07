---
book_chapter: "0410"
book_chapter_name: "Readers"
book_name: Golang入门指南
date: "2016-02-26T17:53:43.0528864+08:00"
description: ""
disqus_identifier: book000104010
slug: ""
title: Golang入门指南-Readers
codeurl: "https://wide.b3log.org/playground/157126313040135745466593b6f65508.go"
---

`io` 包指定了 `io.Reader` 接口，
它表示从数据流结尾读取。

Go 标准库包含了这个接口的[许多实现](https://go-zh.org/search?q=Read#Global)，
包括文件、网络连接、压缩、加密等等。

`io.Reader` 接口有一个 `Read` 方法：

	func (T) Read(b []byte) (n int, err error)

`Read` 用数据填充指定的字节 slice，并且返回填充的字节数和错误信息。
在遇到数据流结尾时，返回 `io.EOF` 错误。

例子代码创建了一个[`strings.Reader`](https://go-zh.org/pkg/strings/#Reader)。
并且以每次 8 字节的速度读取它的输出。

```Go
package main

import (
	"fmt"
	"io"
	"strings"
)

func main() {
	r := strings.NewReader("Hello, Reader!")

	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}

```

