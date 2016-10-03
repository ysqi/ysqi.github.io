---
book_chapter: "0151"
book_chapter_name: "随机数"
book_name: "Go示例大全"
date: "2016-03-04T13:13:33+08:00"
description: ""
disqus_identifier: "book000201051"
slug: "random-numbers"
title: "Go示例大全-随机数"
codeurl: "https://wide.b3log.org/playground/1abcd6cb78cb633afc4656ff45fedcec.go"
---
 
Go 的 `math/rand` 包提供了[伪随机数生成器（英）](http://en.wikipedia.org/wiki/Pseudorandom_number_generator)。







例如，`rand.Intn` 返回一个随机的整数 n，`0 <= n <= 100`。

`rand.Float64` 返回一个64位浮点数 `f`，`0.0 <= f <= 1.0`。

这个技巧可以用来生成其他范围的随机浮点数，例如`5.0 <= f <= 10.0`

默认情况下，给定的种子是确定的，每次都会产生相同的随机数数字序列。要产生变化的序列，需要给定一个变化的种子。需要注意的是，如果你出于加密目的，需要使用随机数的话，请使用 `crypto/rand` 包，此方法不够安全。

调用上面返回的 `rand.Source` 的函数和调用 `rand` 包中函数是相同的。

如果使用相同的种子生成的随机数生成器，将会产生相同的随机数序列。
 

```go
package main  
import "time"
import "fmt"
import "math/rand"  
 func main() {  
 
    fmt.Print(rand.Intn(100), ",")
    fmt.Print(rand.Intn(100))
    fmt.Println()  
 
    fmt.Println(rand.Float64())  
 
    fmt.Print((rand.Float64()*5)+5, ",")
    fmt.Print((rand.Float64() * 5) + 5)
    fmt.Println()  
 
    s1 := rand.NewSource(time.Now().UnixNano())
    r1 := rand.New(s1)  
 
    fmt.Print(r1.Intn(100), ",")
    fmt.Print(r1.Intn(100))
    fmt.Println()  
 
    s2 := rand.NewSource(42)
    r2 := rand.New(s2)
    fmt.Print(r2.Intn(100), ",")
    fmt.Print(r2.Intn(100))
    fmt.Println()
    s3 := rand.NewSource(42)
    r3 := rand.New(s3)
    fmt.Print(r3.Intn(100), ",")
    fmt.Print(r3.Intn(100))
}  
```
