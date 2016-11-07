---
book_chapter: "-"
book_chapter_name: "欢迎"
book_name: Golang入门指南
date: 2016-02-25T15:00:35+08:00
description: ""
disqus_identifier: book00010001
slug: "home"
title: 欢迎
codeurl: https://wide.b3log.org/playground/a6087de933207e84f74cd80605bb83a2.go
---

欢迎来到Go 编程语言指南。

点击页面右边的章节可以访问该指南的模块列表。

任何时候，都可以通过点击页面左边的章节来访问你想看的内容。

在指南后有若干个练习需要读者完成。
 

该指南可以进行交互。点击 “运行”按钮可以在远程服务器上 编译并执行程序。 结果展示在代码的下面。

这些例子程序展示了 Go 的各个方面。在指南中的程序可以成为你积累经验的开始。

编辑程序并且再次执行它。

注意当你点击格式化或运行时，编辑器中的文本会被 gofmt 工具格式化。 


### 关于远程服务器

本指南依托于国内的 Go 在线服务商[b3log](https://wide.b3log.org)来远程执行 Go 程序。该 playground 支持 运行、格式化、分享、讨论 Go 代码。为了保证远程服务器安全，只能编辑调用 Go 官方包，同时会有网络访问限制。


如果你准备好了，请点击页面底部的[下节]进行学习。


```Go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Println("Hello, Go 爱好者！")
	fmt.Println("当前运行时 Go 版本:" + runtime.Version())
}
```