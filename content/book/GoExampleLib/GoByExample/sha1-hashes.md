---
book_chapter: "0154"
book_chapter_name: "SHA1散列"
book_name: "Go示例大全"
date: "2016-03-04T13:13:34+08:00"
description: ""
disqus_identifier: "book000201054"
slug: "sha1-hashes"
title: "Go示例大全-SHA1散列"
codeurl: "https://wide.b3log.org/playground/461427f1fae6739fd54e999cfb3de4dd.go"
---
 
[_SHA1 散列_](http://en.wikipedia.org/wiki/SHA-1)经常用生成二进制文件或者文本块的短标识。例如，[git 版本控制系统](http://git-scm.com/)大量的使用 SHA1 来标识受版本控制的文件和目录。这里是 Go中如何进行 SHA1 散列计算的例子。



Go 在多个 `crypto/*` 包中实现了一系列散列函数。



产生一个散列值得方式是 `sha1.New()`，`sha1.Write(bytes)`，然后 `sha1.Sum([]byte{})`。这里我们从一个新的散列开始。

写入要处理的字节。如果是一个字符串，需要使用`[]byte(s)` 来强制转换成字节数组。

这个用来得到最终的散列值的字符切片。`Sum` 的参数可以用来都现有的字符切片追加额外的字节切片：一般不需要要。

SHA1 值经常以 16 进制输出，例如在 git commit 中。使用`%x` 来将散列结果格式化为 16 进制字符串。
 

```Go
package main  
 
import "crypto/sha1"
import "fmt"  
 func main() {
    s := "sha1 this string"  
 
    h := sha1.New()  
 
    h.Write([]byte(s))  
 
    bs := h.Sum(nil)  
 
    fmt.Println(s)
    fmt.Printf("%x\n", bs)
}  
```
