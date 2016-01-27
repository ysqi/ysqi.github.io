+++
date = "2016-01-27"
title = "Go开发规范之常见问题汇总"
slug = "common_problems-in-Go_development_specifications"
topics = ["开发"]
tags = ["Go","指南"]
disqus_identifier= "2016001"
draft = "true"
+++

这篇文章是Golang官方在Review代码时发现的一些常见问题，也是讨论比较多的，应此罗列这些常见问题供大家参考，这仅仅是一份共识，而不是一份规范指引。可以当作一份GO开发规范的补充信息。

### goimports

虽然Go默认带有 gofmt 工具，但还是强烈推荐一个增强性工具 [goimport] ,该工具在 gofmt 工具基础上增强提供自动删除和引入包功能。

你可在保存文件时，让编辑器自动执行命令，如 Sublime 编辑器插件 `GoSublime` 和 Atom 编辑器插件 `go-plus`均可实现。

当然在使用前，你需要获取包到本地：
```bash
$ go get golang.org/x/tools/cmd/goimports
```

+ 在 Sublime 中使用：具体参见: [插件说明文档](http://michaelwhatcott.com/gosublime-goimports)
+ 在 LiteIDE 中使用：默认支持，如果不起作用，可手工配置：属性配置 -> golangfmt -> `勾选`goimports
+ 在 Atom 中使用：具体参见：[插件说明文档](https://atom.io/packages/go-plus)

### 注释

注释文字应该是一个完整的句子，这有利于在 `godoc` 文档中能看到有效的说名文字。既然是完整的句子，就应该有始有终，末尾以`.`结束。如：
```go
// A Request represents a request to run a command.
type Request struct { ...

// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

### 定义空Slices
当定义一个空的 Slices 时，应该写成：
```go
var t []string
```
而不是：
```go
t := []string{}
```
这样的好处是避免在正式使用前分配内存。

### 文档说明

作为最顶级的，可以供包外部使用的信息均应该写文档说明。

### 不要Painc

不要经常性使用`Panic` ,对于普通的错误信息，应该更多的使用包含 `error` 的多值返回。关于 Error 使用，可参见[官方文档](http://golang.org/doc/effective_go.html#errors)。

### 错误信息

Error 错误信息作为和上下文相关的信息不需要大写(除非是特定名词),也不需要以`.`结束。如使用`fmt.Errorf("something bad")` 而不写成：`fmt.Errorf("Something bad.")`,这样在结合 log 使用时便有完整的信息：`log.Print("Reading %s: %v", filename, err)`。

### 错误处理

不要使用`_`来忽略错误信息，如果一个方法有返回 error，则必须对 error 处理，已确保方法执行成功。处理 Error 时可以返回给上一层，也可以在特殊场景下 `panic`

### 导入包

对于导入的报，应该有组织的分组，组与组间用空行隔开。标准包放在最前面。
```go
package main

import (
    "fmt"
    "hash/adler32"
    "os"

    "appengine/foo"
    "appengine/user"

    "code.google.com/p/x/y"
    "github.com/ysqi/beego"
)
```
高兴的是，goimprots 工具可以自动处理分组。

[goimport]: https://github.com/bradfitz/goimports
