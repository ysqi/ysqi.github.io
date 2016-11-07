---
book_chapter: "0149"
book_chapter_name: "时间戳"
book_name: "Go示例大全"
date: "2016-03-04T13:13:32+08:00"
description: ""
disqus_identifier: "book000201049"
slug: "epoch"
title: "Go示例大全-时间戳"
codeurl: "https://wide.b3log.org/playground/d194e03b7a8134efff491f21741e01b9.go"
---
 
一般程序会有获取 [Unix 时间](http://zh.wikipedia.org/wiki/UNIX%E6%97%B6%E9%97%B4)的秒数，毫秒数，或者微秒数的需要。来看看如何用 Go 来实现。







分别使用带 `Unix` 或者 `UnixNano` 的 `time.Now`来获取从自[协调世界时](http://zh.wikipedia.org/wiki/%E5%8D%94%E8%AA%BF%E4%B8%96%E7%95%8C%E6%99%82)起到现在的秒数或者纳秒数。

注意 `UnixMillis` 是不存在的，所以要得到毫秒数的话，你要自己手动的从纳秒转化一下。

你也可以将协调世界时起的整数秒或者纳秒转化到相应的时间。
 

```Go
package main  
import "fmt"
import "time"  
 func main() {  
 
    now := time.Now()
    secs := now.Unix()
    nanos := now.UnixNano()
    fmt.Println(now)  
 
    millis := nanos / 1000000
    fmt.Println(secs)
    fmt.Println(millis)
    fmt.Println(nanos)  
 
    fmt.Println(time.Unix(secs, 0))
    fmt.Println(time.Unix(0, nanos))
}  
```
