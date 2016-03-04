---
book_chapter: "0153"
book_chapter_name: "URL解析"
book_name: "Go示例大全"
date: "2016-03-04T13:13:34+08:00"
description: ""
disqus_identifier: "book000201053"
slug: "url-parsing"
title: "Go示例大全-URL解析"
codeurl: "https://wide.b3log.org/playground/8981ac7597af3c86b9647d2ad41ecdc3.go"
---
 
URL 提供了一个[统一资源定位方式](http://adam.heroku.com/past/2010/3/30/urls_are_the_uniform_way_to_locate_resources/)。这里了解一下 Go 中是如何解析 URL 的。







我们将解析这个 URL 示例，它包含了一个 scheme，认证信息，主机名，端口，路径，查询参数和片段。

解析这个 URL 并确保解析没有出错。

直接访问 scheme。

`User` 包含了所有的认证信息，这里调用 `Username`和 `Password` 来获取独立值。

`Host` 同时包括主机名和端口信息，如过端口存在的话，使用 `strings.Split()` 从 `Host` 中手动提取端口。

这里我们提出路径和查询片段信息。

要得到字符串中的 `k=v` 这种格式的查询参数，可以使用 `RawQuery` 函数。你也可以将查询参数解析为一个map。已解析的查询参数 map 以查询字符串为键，对应值字符串切片为值，所以如何只想得到一个键对应的第一个值，将索引位置设置为 `[0]` 就行了。
 

```go
package main  
import "fmt"
import "net/url"
import "strings"  
 func main() {  
 
    s := "postgres://user:pass@host.com:5432/path?k=v#f"  
 
    u, err := url.Parse(s)
    if err != nil {
        panic(err)
    }  
 
    fmt.Println(u.Scheme)  
 
    fmt.Println(u.User)
    fmt.Println(u.User.Username())
    p, _ := u.User.Password()
    fmt.Println(p)  
 
    fmt.Println(u.Host)
    h := strings.Split(u.Host, ":")
    fmt.Println(h[0])
    fmt.Println(h[1])  
 
    fmt.Println(u.Path)
    fmt.Println(u.Fragment)  
 
    fmt.Println(u.RawQuery)
    m, _ := url.ParseQuery(u.RawQuery)
    fmt.Println(m)
    fmt.Println(m["k"][0])
}  
```
