---
book_chapter: "0144"
book_chapter_name: "字符串函数"
book_name: "Go示例大全"
date: "2016-03-04T13:13:30+08:00"
description: ""
disqus_identifier: "book000201044"
slug: "string-functions"
title: "Go示例大全-字符串函数"
codeurl: "https://wide.b3log.org/playground/2a599f3a5b4dee503470f5fa2bef9e41.go"
---
 
标准库的 `strings` 包提供了很多有用的字符串相关的函数。这里是一些用来让你对这个包有个初步了解的例子。





我们给 `fmt.Println` 一个短名字的别名，我们随后将会经常用到。



这是一些 `strings` 中的函数例子。注意他们都是包中的函数，不是字符串对象自身的方法，这意味着我们需要考虑在调用时传递字符作为第一个参数进行传递。

你可以在 [`strings`](http://golang.org/pkg/strings/)包文档中找到更多的函数

虽然不是 `strings` 的一部分，但是仍然值得一提的是获取字符串长度和通过索引获取一个字符的机制。
 

```go
package main  
import s "strings"
import "fmt"  
 
var p = fmt.Println  
 func main() {  
 
    p("Contains:  ", s.Contains("test", "es"))
    p("Count:     ", s.Count("test", "t"))
    p("HasPrefix: ", s.HasPrefix("test", "te"))
    p("HasSuffix: ", s.HasSuffix("test", "st"))
    p("Index:     ", s.Index("test", "e"))
    p("Join:      ", s.Join([]string{"a", "b"}, "-"))
    p("Repeat:    ", s.Repeat("a", 5))
    p("Replace:   ", s.Replace("foo", "o", "0", -1))
    p("Replace:   ", s.Replace("foo", "o", "0", 1))
    p("Split:     ", s.Split("a-b-c-d-e", "-"))
    p("ToLower:   ", s.ToLower("TEST"))
    p("ToUpper:   ", s.ToUpper("test"))
    p()  
 
 
    p("Len: ", len("hello"))
    p("Char:", "hello"[1])
}  
```
