---
book_chapter: "0121"
book_chapter_name: "错误处理"
book_name: "Go示例大全"
date: "2016-03-04T13:13:21+08:00"
description: ""
disqus_identifier: "book000201021"
slug: "errors"
title: "Go示例大全-错误处理"
codeurl: "https://wide.b3log.org/playground/29782288e8e697eb5ae9d9c0f4334ae2.go"
---
 
Go 语言使用一个独立的·明确的返回值来传递错误信息的。这与使用异常的 Java 和 Ruby 以及在 C 语言中经常见到的超重的单返回值/错误值相比，Go 语言的处理方式能清楚的知道哪个函数返回了错误，并能想调用那些没有出错的函数一样调用。





按照惯例，错误通常是最后一个返回值并且是 `error` 类型，一个内建的接口。

`errors.New` 构造一个使用给定的错误信息的基本`error` 值。



返回错误值为 nil 代表没有错误。

通过实现 `Error` 方法来自定义 `error` 类型是可以得。这里使用自定义错误类型来表示上面的参数错误。





在这个例子中，我们使用 `&argError` 语法来建立一个新的结构体，并提供了 `arg` 和 `prob` 这个两个字段的值。



下面的两个循环测试了各个返回错误的函数。注意在 `if`行内的错误检查代码，在 Go 中是一个普遍的用法。

你如果想在程序中使用一个自定义错误类型中的数据，你需要通过类型断言来得到这个错误类型的实例。
 

```go
package main  
import "errors"
import "fmt"  
 
func f1(arg int) (int, error) {
    if arg == 42 {  
 
        return -1, errors.New("can't work with 42")  
     }  
 
    return arg + 3, nil
}  
 
type argError struct {
    arg  int
    prob string
}  
 func (e *argError) Error() string {
    return fmt.Sprintf("%d - %s", e.arg, e.prob)
}  
 func f2(arg int) (int, error) {
    if arg == 42 {  
 
        return -1, &argError{arg, "can't work with it"}
    }
    return arg + 3, nil
}  
 func main() {  
 
    for _, i := range []int{7, 42} {
        if r, e := f1(i); e != nil {
            fmt.Println("f1 failed:", e)
        } else {
            fmt.Println("f1 worked:", r)
        }
    }
    for _, i := range []int{7, 42} {
        if r, e := f2(i); e != nil {
            fmt.Println("f2 failed:", e)
        } else {
            fmt.Println("f2 worked:", r)
        }
    }  
 
    _, e := f2(42)
    if ae, ok := e.(*argError); ok {
        fmt.Println(ae.arg)
        fmt.Println(ae.prob)
    }
}  
```
