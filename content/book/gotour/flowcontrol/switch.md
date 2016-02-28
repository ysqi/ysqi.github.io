---
book_chapter: "0209"
book_chapter_name: "switch"
book_name: Golang入门指南
date: "2016-02-26T17:45:29.7906734+08:00"
description: ""
disqus_identifier: book00010209
slug: ""
title: Golang入门指南-switch
codeurl: "https://wide.b3log.org/playground/.go"
---




你可能已经知道 `switch` 语句会长什么样了。

除非以 `fallthrough` 语句结束，否则分支会自动终止。

```
// +build OMIT

package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.", os)
	}
}

```

