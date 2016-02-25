---
book_chapter: "1.2"
book_chapter_name: "导入包"
book_name: Golang入门指南  
date: 2016-02-25T15:51:15+08:00
description: ""
disqus_identifier: book00010103 
title: 导入包 
codeurl: https://wide.b3log.org/playground/5737c4c43872d9f28696d037f7e5d4aa.go
---

这个代码用圆括号组合了导入，这是**打包**导入语句。

同样可以编写多个导入语句，例如：
```go
import "fmt"
import "math/rand"
```
不过使用打包的导入语句是更好的形式。

### 最佳实践
对于导入包的推荐一种最合适的导入形式。

对导入包进行特定分组，按标准包, 项目包，第三方包 依次分组，组与组间用空行隔开，其中标准包放在最前面。

例如:
```go
package main
import (
    "fmt"
    "hash/adler32"
    "os"

    "appengine/foo"
    "appengine/user"

    "code.google.com/p/x/y"
    "github.com/ysqi/beego"
)
```