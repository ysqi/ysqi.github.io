---
book_chapter: "0412"
book_chapter_name: "练习：rot13Reader"
book_name: Golang入门指南
date: "2016-02-26T17:53:43.9349368+08:00"
description: ""
disqus_identifier: book000104012
slug: "exercise-rot-reader"
title: Golang入门指南-练习：rot13Reader
codeurl: "https://wide.b3log.org/playground/.go"
---




一个常见模式是 [[https://go-zh.org/pkg/io/#Reader][io.Reader]] 
包裹另一个 `io.Reader`，然后通过某种形式修改数据流。

例如，[[https://go-zh.org/pkg/compress/gzip/#NewReader][gzip.NewReader]] 
函数接受 `io.Reader`（压缩的数据流）并且返回同样实现了 `io.Reader` 的 `*gzip.Reader`（解压缩后的数据流）。

编写一个实现了 `io.Reader` 的 `rot13Reader`，
并从一个 `io.Reader` 读取，
利用 [[https://en.wikipedia.org/wiki/ROT13][rot13]] 代换密码对数据流进行修改。

已经帮你构造了 `rot13Reader` 类型。
通过实现 `Read` 方法使其匹配 `io.Reader`。

```
// +build OMIT

package main

import (
	"io"
	"os"
	"strings"
)

type rot13Reader struct {
	r io.Reader
}

func main() {
	s := strings.NewReader("Lbh penpxrq gur pbqr!")
	r := rot13Reader{s}
	io.Copy(os.Stdout, &r)
}

```

