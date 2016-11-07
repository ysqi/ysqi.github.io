---
book_chapter: "0117"
book_chapter_name: "指针"
book_name: "Go示例大全"
date: "2016-03-04T13:13:20+08:00"
description: ""
disqus_identifier: "book000201017"
slug: "pointers"
title: "Go示例大全-指针"
codeurl: "https://wide.b3log.org/playground/71a644ed8034815c49df3aa6606ed95f.go"
---
 
Go 支持 <em><a href="http://zh.wikipedia.org/wiki/%E6%8C%87%E6%A8%99_(%E9%9B%BB%E8%85%A6%E7%A7%91%E5%AD%B8)">指针</a></em>，允许在程序中通过引用传递值或者数据结构。





我们将通过两个函数：`zeroval` 和 `zeroptr` 来比较指针和值类型的不同。`zeroval` 有一个 `int` 型参数，所以使用值传递。`zeroval` 将从调用它的那个函数中得到一个 `ival`形参的拷贝。

`zeroptr` 有一和上面不同的 `*int` 参数，意味着它用了一个 `int`指针。函数体内的 `*iptr` 接着_解引用_ 这个指针，从它内存地址得到这个地址对应的当前值。对一个解引用的指针赋值将会改变这个指针引用的真实地址的值。





通过 `&i` 语法来取得 `i` 的内存地址，例如一个变量`i` 的指针。

指针也是可以被打印的。
 

```Go
package main  
import "fmt"  
 
func zeroval(ival int) {
    ival = 0
}  
 
func zeroptr(iptr *int) {
    *iptr = 0
}  
 func main() {
    i := 1
    fmt.Println("initial:", i)  
     zeroval(i)
    fmt.Println("zeroval:", i)  
 
    zeroptr(&i)
    fmt.Println("zeroptr:", i)  
 
    fmt.Println("pointer:", &i)
}  
```
