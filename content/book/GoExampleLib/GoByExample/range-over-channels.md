---
book_chapter: "0131"
book_chapter_name: "通道遍历"
book_name: "Go示例大全"
date: "2016-03-04T13:13:25+08:00"
description: ""
disqus_identifier: "book000201031"
slug: "range-over-channels"
title: "Go示例大全-通道遍历"
codeurl: "https://wide.b3log.org/playground/2ffa8a78bb0951f70fc3ea4160f917d1.go"
---
 
在[前面](../range/)的例子中，我们讲过 `for` 和 `range`为基本的数据结构提供了迭代的功能。我们也可以使用这个语法来遍历从通道中取得的值。







我们将遍历在 `queue` 通道中的两个值。

这个 `range` 迭代从 `queue` 中得到的每个值。因为我们在前面 `close` 了这个通道，这个迭代会在接收完 2 个值之后结束。如果我们没有 `close` 它，我们将在这个循环中继续阻塞执行，等待接收第三个值
 

```go
package main  
import "fmt"  
 func main() {  
 
    queue := make(chan string, 2)
    queue <- "one"
    queue <- "two"
    close(queue)  
 
    for elem := range queue {
        fmt.Println(elem)
    }
}  
```
