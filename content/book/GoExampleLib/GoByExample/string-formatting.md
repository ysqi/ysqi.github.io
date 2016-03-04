---
book_chapter: "0145"
book_chapter_name: "字符串格式化"
book_name: "Go示例大全"
date: "2016-03-04T13:13:30+08:00"
description: ""
disqus_identifier: "book000201045"
slug: "string-formatting"
title: "Go示例大全-字符串格式化"
codeurl: "https://wide.b3log.org/playground/8f7a422f91cbb5588917bf10851156bb.go"
---
 
Go 在传统的`printf` 中对字符串格式化提供了优异的支持。这里是一些基本的字符串格式化的人物的例子。









Go 为常规 Go 值的格式化设计提供了多种打印方式。例如，这里打印了 `point` 结构体的一个实例。

如果值是一个结构体，`%+v` 的格式化输出内容将包括结构体的字段名。

`%#v` 形式则输出这个值的 Go 语法表示。例如，值的运行源代码片段。

需要打印值的类型，使用 `%T`。

格式化布尔值是简单的。

格式化整形数有多种方式，使用 `%d`进行标准的十进制格式化。

这个输出二进制表示形式。

这个输出给定整数的对应字符。

`%x` 提供十六进制编码。

对于浮点型同样有很多的格式化选项。使用 `%f` 进行最基本的十进制格式化。

`%e` 和 `%E` 将浮点型格式化为（稍微有一点不同的）科学技科学记数法表示形式。

使用 `%s` 进行基本的字符串输出。

像 Go 源代码中那样带有双引号的输出，使用 `%q`。

和上面的整形数一样，`%x` 输出使用 base-16 编码的字符串，每个字节使用 2 个字符表示。

要输出一个指针的值，使用 `%p`。

当输出数字的时候，你将经常想要控制输出结果的宽度和精度，可以使用在 `%` 后面使用数字来控制输出宽度。默认结果使用右对齐并且通过空格来填充空白部分。

你也可以指定浮点型的输出宽度，同时也可以通过 宽度.精度 的语法来指定输出的精度。

要左对齐，使用 `-` 标志。

你也许也想控制字符串输出时的宽度，特别是要确保他们在类表格输出时的对齐。这是基本的右对齐宽度表示。

要左对齐，和数字一样，使用 `-` 标志。

到目前为止，我们已经看过 `Printf`了，它通过 `os.Stdout`输出格式化的字符串。`Sprintf` 则格式化并返回一个字符串而不带任何输出。

你可以使用 `Fprintf` 来格式化并输出到 `io.Writers`而不是 `os.Stdout`。
 

```go
package main  
import "fmt"
import "os"  
 type point struct {
    x, y int
}  
 func main() {  
 
    p := point{1, 2}
    fmt.Printf("%v\n", p)  
 
    fmt.Printf("%+v\n", p)  
 
    fmt.Printf("%#v\n", p)  
 
    fmt.Printf("%T\n", p)  
 
    fmt.Printf("%t\n", true)  
 
    fmt.Printf("%d\n", 123)  
 
    fmt.Printf("%b\n", 14)  
 
    fmt.Printf("%c\n", 33)  
 
    fmt.Printf("%x\n", 456)  
 
    fmt.Printf("%f\n", 78.9)  
 
    fmt.Printf("%e\n", 123400000.0)
    fmt.Printf("%E\n", 123400000.0)  
 
    fmt.Printf("%s\n", "\"string\"")  
 
    fmt.Printf("%q\n", "\"string\"")  
 
    fmt.Printf("%x\n", "hex this")  
 
    fmt.Printf("%p\n", &p)  
 
    fmt.Printf("|%6d|%6d|\n", 12, 345)  
 
    fmt.Printf("|%6.2f|%6.2f|\n", 1.2, 3.45)  
 
    fmt.Printf("|%-6.2f|%-6.2f|\n", 1.2, 3.45)  
 
    fmt.Printf("|%6s|%6s|\n", "foo", "b")  
 
    fmt.Printf("|%-6s|%-6s|\n", "foo", "b")  
 
    s := fmt.Sprintf("a %s", "string")
    fmt.Println(s)  
 
    fmt.Fprintf(os.Stderr, "an %s\n", "error")
}  
```
