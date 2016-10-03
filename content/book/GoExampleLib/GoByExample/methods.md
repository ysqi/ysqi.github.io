---
book_chapter: "0119"
book_chapter_name: "方法"
book_name: "Go示例大全"
date: "2016-03-04T13:13:21+08:00"
description: ""
disqus_identifier: "book000201019"
slug: "methods"
title: "Go示例大全-方法"
codeurl: "https://wide.b3log.org/playground/c230acaede77234b0adb61fd5d4af1ba.go"
---
 
Go 支持在结构体类型中定义_方法_ 。







这里的 `area` 方法有一个_接收器类型_ `rect`。

可以为值类型或者指针类型的接收器定义方法。这里是一个值类型接收器的例子。



这里我们调用上面为结构体定义的两个方法。

Go 自动处理方法调用时的值和指针之间的转化。你可以使用指针来调用方法来避免在方法调用时产生一个拷贝，或者让方法能够改变接受的数据。
 

```go
package main  
import "fmt"  
 type rect struct {
    width, height int
}  
 
func (r *rect) area() int {
    return r.width * r.height
}  
 
func (r rect) perim() int {
    return 2*r.width + 2*r.height
}  
 func main() {
    r := rect{width: 10, height: 5}  
 
    fmt.Println("area: ", r.area())
    fmt.Println("perim:", r.perim())  
 
    rp := &r
    fmt.Println("area: ", rp.area())
    fmt.Println("perim:", rp.perim())
}  
```
