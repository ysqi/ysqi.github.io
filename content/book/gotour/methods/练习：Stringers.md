---
book_chapter: "4.7"
book_chapter_name: "练习：Stringers"
book_name: Golang入门指南
date: "2016-02-26T17:53:41.1017748+08:00"
description: ""
disqus_identifier: book00010407
slug: ""
title: Golang入门指南-练习：Stringers
codeurl: "https://wide.b3log.org/playground/.go"
---




让 `IPAddr` 类型实现 `fmt.Stringer` 以便用点分格式输出地址。

例如，`IPAddr{1,`2,`3,`4}` 应当输出 `"1.2.3.4"`。

```
package main

import "fmt"

type IPAddr [4]byte

// TODO: Add a "String() string" method to IPAddr.

func main() {
	addrs := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for n, a := range addrs {
		fmt.Printf("%v: %v\n", n, a)
	}
}

```

