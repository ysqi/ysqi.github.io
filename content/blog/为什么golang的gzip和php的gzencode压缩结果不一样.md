---
date: 2015-07-31
title: 为什么golang的gzip和php的gzencode压缩结果不一样
slug: golang-php-gzencode-difrent
tags : 
- gzip
- 问题
- Go
- PHP
Topics: 
- 开发
description: 不同语言，不同算法，结果有所不同，但最重要的是，数据能够被解压，再不同语言间相互解压，则就满足了，所以不需要太关注为什么Go和PHP的Gzip压缩结果不一样
---

一次功能需要接对百度统计dataApi，根据百度提供的PHP Demo，写golang的实现，但是老提示数据格式错误，数据已损坏，一路分析，解决了不少问题，其中对golang和php的gzip压缩结果不一样产生了疑问，各种求助，最终知道数据不一样是正常的。


**首先看源代码**

```go
package main

import (
	"bytes"
	"compress/gzip"
	"fmt"
)

func main() {

	data := "a"

	buffer := new(bytes.Buffer)
	w, _ := gzip.NewWriterLevel(buffer, 9) //php: gzencode($json_data,9)
	defer w.Close()
	w.Write([]byte(data))
	w.Flush()

	fmt.Println(buffer.Bytes()[:])
}
```

不管是哪种语言的压缩，其实基本上都是基于标准的，而Golang和PHP都是基于[RFC 1952](https://tools.ietf.org/html/rfc1952) ,gzip数据格式如下：
```
+---+---+---+---+---+---+---+---+---+---+

|ID1|ID2|CM |FLG|  MTIME        |XFL|OS | (more-->)

+---+---+---+---+---+---+---+---+---+---+
```
再对比PHP和GoLang的头部定义

|-                         |     PHP   |     Go |
|--------------------------|-----------|---------|
|D1                        |      31   |     31
|D2                        |     139   |     139
|CM (compression method)   |       8   |     8
|FLG (flags)               |       0   |     0
|MTIME (modification time) |  0 0 0 0  |  0 9 110 136
|XFL (extra flags)         |       0   |      0
|OS (operating system)     |       0   |      255


上面看到，Go 设置的头信息和PHP有部分差役，Go都设置了MTime（修改时间） 和OS（操作系统，为255 ，不知道代表什么意思），而PHP中的OS＝0表示是 FAT文件系统。

### 有差役是正常的，重要的是能相互解压出数据

不同语言，默认的压缩级别不一样，也会导致压缩结果不一样，Go中默认级别是-1(DefaultCompression)。

```go
func NewWriter(w io.Writer) *Writer {
    z, _ := NewWriterLevel(w, DefaultCompression)
    return z
}
```

虽然我们设置了和PHP中一样的压缩级别，但是结果依旧可能还是不一样，这是因为GZIP 软件核心算法deflate，是LZ77和Huffman压缩的结合，所使用的频率树来决定输出码。

### 最重要的压缩数据能够被正确的解压

不同语言，不同算法，结果有所不同，但最重要的是，数据能够被解压，再不同语言间相互解压，则就满足了，所以不需要太关注为什么Go和PHP的Gzip压缩结果不一样。
