---
book_chapter: "0141"
book_chapter_name: "Panic"
book_name: "Go示例大全"
date: "2016-03-04T13:13:29+08:00"
description: ""
disqus_identifier: "book000201041"
slug: "panic"
title: "Go示例大全-Panic"
codeurl: "https://wide.b3log.org/playground/dead5888864f0a758f696f7defc57cb5.go"
---
 
`panic` 意味着有些出乎意料的错误发生。通常我们用它来表示程序正常运行中不应该出现的，后者我么没有处理好的错误。







我们将在真个网站中使用 panic 来检查预期外的错误。这个是唯一一个为 panic 准备的例子。

panic 的一个基本用法就是在一个函数返回了错误值但是我们并不知道（或者不想）处理时终止运行。这里是一个在创建一个新文件时返回异常错误时的`panic` 用法。
 

```Go
package main  
import "os"  
 func main() {  
 
    panic("a problem")  
 
    _, err := os.Create("/tmp/file")
    if err != nil {
        panic(err)
    }
}  
```
