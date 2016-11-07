---
book_chapter: "0150"
book_chapter_name: "时间的格式化和解析"
book_name: "Go示例大全"
date: "2016-03-04T13:13:32+08:00"
description: ""
disqus_identifier: "book000201050"
slug: "time-formatting-parsing"
title: "Go示例大全-时间的格式化和解析"
codeurl: "https://wide.b3log.org/playground/f3dd2c0f7206203b0a5b2bfe4f98bcf7.go"
---
 
Go 支持通过基于描述模板的时间格式化和解析。







这里是一个基本的按照 RFC3339 进行格式化的例子，使用对应模式常量。

时间解析使用同 `Format` 相同的形式值。

`Format` 和 `Parse` 使用基于例子的形式来决定日期格式，一般你只要使用 `time` 包中提供的模式常量就行了，但是你也可以实现自定义模式。模式必须使用时间 `Mon Jan 2 15:04:05 MST 2006`来指定给定时间/字符串的格式化/解析方式。时间一定要按照如下所示：2006为年，15 为小时，Monday 代表星期几，等等。

对于纯数字表示的时间，你也可以使用标准的格式化字符串来提出出时间值得组成。

`Parse` 函数在输入的时间格式不正确是会返回一个错误。
 

```Go
package main  
import "fmt"
import "time"  
 func main() {
    p := fmt.Println  
 
    t := time.Now()
    p(t.Format(time.RFC3339))  
 
    t1, e := time.Parse(
        time.RFC3339,
        "2012-11-01T22:08:41+00:00")
    p(t1)  
 
    p(t.Format("3:04PM"))
    p(t.Format("Mon Jan _2 15:04:05 2006"))
    p(t.Format("2006-01-02T15:04:05.999999-07:00"))
    form := "3 04 PM"
    t2, e := time.Parse(form, "8 41 PM")
    p(t2)  
 
    fmt.Printf("%d-%02d-%02dT%02d:%02d:%02d-00:00\n",
        t.Year(), t.Month(), t.Day(),
        t.Hour(), t.Minute(), t.Second())  
 
    ansic := "Mon Jan _2 15:04:05 2006"
    _, e = time.Parse(ansic, "8:41PM")
    p(e)
}  
```
