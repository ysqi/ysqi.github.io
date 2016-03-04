---
book_chapter: "0148"
book_chapter_name: "时间"
book_name: "Go示例大全"
date: "2016-03-04T13:13:32+08:00"
description: ""
disqus_identifier: "book000201048"
slug: "time"
title: "Go示例大全-时间"
codeurl: "https://wide.b3log.org/playground/1f4a0bcd87970ab4053b734b0771d63d.go"
---
 
Go 对时间和时间段提供了大量的支持；这里是一些例子。







得到当前时间。

通过提供年月日等信息，你可以构建一个 `time`。时间总是关联着位置信息，例如时区。

你可以提取出时间的各个组成部分。

输出是星期一到日的 `Weekday` 也是支持的。

这些方法来比较两个时间，分别测试一下是否是之前，之后或者是同一时刻，精确到秒。

方法 `Sub` 返回一个 `Duration` 来表示两个时间点的间隔时间。

我们计算出不同单位下的时间长度值。

你可以使用 `Add` 将时间后移一个时间间隔，或者使用一个 `-` 来将时间前移一个时间间隔。
 

```go
package main  
import "fmt"
import "time"  
 func main() {
    p := fmt.Println  
 
    now := time.Now()
    p(now)  
 
    then := time.Date(
        2009, 11, 17, 20, 34, 58, 651387237, time.UTC)
    p(then)  
 
    p(then.Year())
    p(then.Month())
    p(then.Day())
    p(then.Hour())
    p(then.Minute())
    p(then.Second())
    p(then.Nanosecond())
    p(then.Location())  
 
    p(then.Weekday())  
 
    p(then.Before(now))
    p(then.After(now))
    p(then.Equal(now))  
 
    diff := now.Sub(then)
    p(diff)  
 
    p(diff.Hours())
    p(diff.Minutes())
    p(diff.Seconds())
    p(diff.Nanoseconds())  
 
    p(then.Add(diff))
    p(then.Add(-diff))
}  
```
