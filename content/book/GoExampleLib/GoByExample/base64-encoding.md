---
book_chapter: "0155"
book_chapter_name: "Base64编码"
book_name: "Go示例大全"
date: "2016-03-04T13:13:34+08:00"
description: ""
disqus_identifier: "book000201055"
slug: "base64-encoding"
title: "Go示例大全-Base64编码"
codeurl: "https://wide.b3log.org/playground/643495c6ab36e0c72aa798b0a837fea0.go"
---
 
Go 提供内建的 [base64 编解码](http://zh.wikipedia.org/wiki/Base64)支持。



这个语法引入了 `encoding/base64` 包并使用名称 `b64`代替默认的 `base64`。这样可以节省点空间。



这是将要编解码的字符串。

Go 同时支持标准的和 URL 兼容的 base64 格式。编码需要使用 `[]byte` 类型的参数，所以要将字符串转成此类型。

解码可能会返回错误，如果不确定输入信息格式是否正确，那么，你就需要进行错误检查了。

使用 URL 兼容的 base64 格式进行编解码。
 

```Go
package main  
 
import b64 "encoding/base64"
import "fmt"  
 func main() {  
 
    data := "abc123!?$*&()'-=@~"  
 
    sEnc := b64.StdEncoding.EncodeToString([]byte(data))
    fmt.Println(sEnc)  
 
    sDec, _ := b64.StdEncoding.DecodeString(sEnc)
    fmt.Println(string(sDec))
    fmt.Println()  
 
    uEnc := b64.URLEncoding.EncodeToString([]byte(data))
    fmt.Println(uEnc)
    uDec, _ := b64.URLEncoding.DecodeString(uEnc)
    fmt.Println(string(uDec))
}  
```
