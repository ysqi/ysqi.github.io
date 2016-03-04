---
book_chapter: "0158"
book_chapter_name: "行过滤器"
book_name: "Go示例大全"
date: "2016-03-04T13:13:36+08:00"
description: ""
disqus_identifier: "book000201058"
slug: "line-filters"
title: "Go示例大全-行过滤器"
codeurl: "https://wide.b3log.org/playground/47951d92a664c25d155bd4e392910663.go"
---
 
一个_行过滤器_ 在读取标准输入流的输入，处理该输入，然后将得到一些的结果输出到标准输出的程序中是常见的一个功能。`grep` 和 `sed` 是常见的行过滤器。

这里是一个使用 Go 编写的行过滤器示例，它将所有的输入文字转化为大写的版本。你可以使用这个模式来写一个你自己的 Go行过滤器。





对 `os.Stdin` 使用一个带缓冲的 scanner，让我们可以直接使用方便的 `Scan` 方法来直接读取一行，每次调用该方法可以让 scanner 读取下一行。

`Text` 返回当前的 token，现在是输入的下一行。



写出大写的行。

检查 `Scan` 的错误。文件结束符是可以接受的，并且不会被 `Scan` 当作一个错误。
 

```go
 
package main  
import (
    "bufio"
    "fmt"
    "os"
    "strings"
)  
 func main() {  
 
    scanner := bufio.NewScanner(os.Stdin)  
     for scanner.Scan() {  
         ucl := strings.ToUpper(scanner.Text())  
 
        fmt.Println(ucl)
    }  
 
    if err := scanner.Err(); err != nil {
        fmt.Fprintln(os.Stderr, "error:", err)
        os.Exit(1)
    }
}  
```
