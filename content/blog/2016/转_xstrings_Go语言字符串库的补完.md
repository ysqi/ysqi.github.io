
---
date: 2016-12-31T11:34:36+08:00
title: "xstrings:Go语言字符串库的补完"
description: ""
disqus_identifier: 1485833676748599675
slug: "xstrings:Go-yu-yan-zi-fu-chuan-ku-de-bu-wan"
source: "https://segmentfault.com/a/1190000002515490"
tags: 
- 算法 
- string 
- 开放源代码 
- golang 
topics:
- 编程语言与开发
---

项目地址：<https://github.com/huandu/xstrings>

`xstrings` 是一个很简单的 Go 语言库，简单说就是提供了一些标准库
`strings`
没提供但依然很有用的字符串算法。每个字符串算法都对效率进行了优化，所有函数都可以做到不超过
`O(n)` 的复杂度，并且尽量节省内存使用，仅在需要分配内存的时候分配。

现在实现的算法几乎都是其他语言（主要是
Python/Ruby/PHP/Perl）标准库里提供的算法，用 Go
重新实现一遍。未来也许我还会继续加入更多的方法，不过我不希望这个库成为一个算法大杂烩，因此仅仅会考虑那些特别有名且语言无关的函数。

Go 的 `strings` 操作字符串的时候都是以 `rune` 为单位进行，但 `string`
类型却只能以 `byte` 为单位进行下标访问，要想使用 `rune` 就得用
`[]rune(str)` 进行转换（有额外内存分配）或者
`utf8. DecodeRuneInString(str)`
一个个解码。为了和标准库保持高度统一，`xstrings` 也完全以 `rune`
为单位操作字符串，这使得算法的实现难度比其他语言稍高一点，不过总之都搞定了，这种有一点挑战的感觉很不错。٩(
'ω' )و

做这个项目有两方面原因。

一方面是因为 Go 的 `strings`
自带算法实在太少，其他语言里面一些特别有趣的字符串算法，比如 Ruby 的
`String#succ`/`String#tr` 在 Go
里面都没有，网上甚至都搜不到有人实现过，所以干脆自己来实现一个。

另一方面则是因为 Go
语言的命名风格比较另类，或者说没有什么历史包袱，在其他语言都被 C
语言遗毒深深困扰的时候，Go
采用了一种小清新的风格命名函数，读起来很舒服，只是从其他语言过来的人要想凭感觉找到某特定函数时会略微有些纠结，比如谁能想到
`stricmp` 在 Go 里面竟然叫做 `EqualFold`
呢。所以我就干脆写一个函数对应表，把 Go 的 `strings`
函数和其他语言类似函数对应起来，也许有朝一日能够帮到些许新晋 Go
开发者吧。

