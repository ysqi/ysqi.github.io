---
book_chapter: "-"
book_chapter_name: "前言"
book_name: Go示例大全
date: 2016-03-04T11:57:22+08:00
description: ""
disqus_identifier: book00020001
slug: "home"
title: 前言
codeurl: https://wide.b3log.org/playground/a6087de933207e84f74cd80605bb83a2.go
---

《Go示例大全》顾名思义便是包罗万千的 Go 示例代码，通过示例代码。Go 开发者/新手能快速掌握 Go 各模块的使用。

目前本电子书还在建立初期，后续会加大 Go 优秀示例收集整理速度。欢迎大家在下方留言推荐示例。


欢迎大家反馈意见，在下发评论区留下你的足迹，是对博主最大的支持！


### 关于远程服务器

本指南依托于国内的 Go 在线服务商[b3log](https://wide.b3log.org)来远程执行 Go 程序。该 playground 支持 运行、格式化、分享、讨论 Go 代码。为了保证远程服务器安全，只能编辑调用 Go 官方包，同时会有网络访问限制。


**注意**为加快示例显示速度，默认是不加载在线编辑环境。用户可以点击按钮【在线运行】来加载编辑环境。


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