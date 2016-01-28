+++
date = "2016-01-27"
title = "Go开发规范之常见问题汇总"
slug = "common_problems-in-Go_development_specifications"
topics = ["开发"]
tags = ["Go","编码规范"]
disqus_identifier= "2016001" 
+++

这篇文章是Golang官方在Review代码时发现的一些常见问题，也是讨论比较多的，应此罗列这些常见问题供大家参考，这仅仅是一份共识，而不是一份规范指引。可以当作一份GO开发规范的补充信息。

### goimports

虽 Go 默认带有 gofmt 工具，但还是强烈推荐增强性工具 [goimport] ,该工具在 gofmt 工具基础上增强提供自动删除和引入包功能。

你可在保存文件时，让编辑器自动执行命令，如 Sublime 编辑器插件 `GoSublime` 和 Atom 编辑器插件 `go-plus`均可实现。

在使用前，你需要获取包到本地：
```bash
$ go get golang.org/x/tools/cmd/goimports
```

+ 在 Sublime 中使用：具体参见: [插件说明文档](http://michaelwhatcott.com/gosublime-goimports)
+ 在 LiteIDE 中使用：默认支持，如果不起作用，可手工配置：属性配置 -> golangfmt -> `勾选`goimports
+ 在 Atom 中使用：具体参见：[插件说明文档](https://atom.io/packages/go-plus)

### 注释

注释应该是一个完整的句子，这有利于在 `godoc` 文档中能看到有效的完整文字。既然是完整的句子，就应该有始有终，末尾以`.`结束。如：
```go
// A Request represents a request to run a command.
type Request struct { ...

// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

### 定义空Slices
当定义一个空的 Slices 时，应该写成：
```go
var t []string
```
而不是：
```go
t := []string{}
```
这样的好处是避免在正式使用前分配内存。

### 文档说明

作为最顶级的，可以供包外部使用的信息均应该写文档说明。

### 不要Painc

不要到处使用`Panic` ,对于普通的错误，应该更多的使用包含 `error` 的多值返回。关于 Error 使用，可参见[官方文档](http://golang.org/doc/effective_go.html#errors)。

### 错误信息

error 错误信息作为和上下文相关的信息不需要大写(除非是特定名词),也不需要以`.`结束。如使用`fmt.Errorf("something bad")` 而不写成：`fmt.Errorf("Something bad.")`,这样在结合 log 使用时便有完整的信息：`log.Print("Reading %s: %v", filename, err)`。

### 错误处理

不要使用`_`来忽略错误信息，如果一个方法有返回 error，则必须对 error 处理，已确保方法执行成功。处理 Error 时可以返回给上一层，也可以在特殊场景下 `panic`

### 导入包

对导入包进行特定分组，按标准包, 项目包，第三方包 依次分组，组与组间用空行隔开，其中标准包放在最前面。
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
高兴的是，goimprots 工具可以自动处理分组。

### 精简错误处理
尽量让代码按常规逻辑处理，先处理错误信息，缩短和精简错误处理范围，有利于提高代码可读性。不要写成：
```go
if err != nil {
    // 错误处理
} else {
    // 常规逻辑
}
```
而应写成：
```go
if err != nil {
    // 错误处理
    return // 或者继续.
}
// 常规逻辑
```
如果包含变量赋值，大部分人会写成：
```go
if x, err := f(); err != nil {
  // 错误处理
  return
} else {
  // 使用变量 x
}
这时可以将赋值放到前面，精简错误处理：
```go
x, err := f();
if  err != nil {
  // 错误处理
  return
} 
// 使用变量 x
``` 

### 命名简称
在使用简称或者缩写 (如："URL" , "RMB" ) 时需保持命名一致性。例如`URL`使用时要么是`URL`，要么是`url`,不应该写出`Url`(如："urlPony" , "URLPony")。Go语言中一个很好的例子是`ServerHTTP`,而不是ServerHttp。

该规则也适合于简称，如`ID`是 identifier 的简称，使用是应该写出`appID`而不是`appId`。

规范命名能有效提高可阅读性。

### 单行长度
这不是 Go 的的硬性规定，但单行长文字给人看得不舒服，如果单号超过越80个字符，请换行。长文字是不利于阅读的，分多行精简内容才是王道。看过报纸就知道，不是一篇文章从报纸最左边写到最右边，而是分成多个小区域排版的。当然Go 和 godoc 作为程序是能很好的展示的。

### 返回参数命名

根据实际情况给返还参数命名，想想看如果在 godoc 中看到：
```go
func (n *Node) Parent1() (node *Node)
func (n *Node) Parent2() (node *Node, err error)
```
这个命名就完全没必要，还不如写成：
```go
func (n *Node) Parent1() *Node
func (n *Node) Parent2() (*Node, error)
```
实际上一两个参数，通过返回类型就能明白含义的话，就没必要给返回参数命名。但另一方面，如果返回参数包含多个相同类型，或者没法推断含义的话，则需要给返回参数命名，如：
```go
func (f *Foo) Location() (float64, float64, error)
```
此时，更好的写法是：
```go
// Location returns f's latitude and longitude.
// Negative values mean south and west, respectively.
func (f *Foo) Location() (lat ,long float64, err error)
```
不管项目规模大写，合理的给返回参数命名和添加注释，以此给项目增强可读性比你节约几个单词更为重要。

### 包注释
包注释是完整的现实在 godoc 中，用于描述此包的用途，可包含包使用指南。注释内容和包之间不能有空行，单号注释使用`//`，多行注释使用`/*  */`包裹。
```go
// Package math provides basic constants and mathematical functions.
package math

/*
Package regexp implements a simple library for regular expressions.

The syntax of the regular expressions accepted is:

    regexp:
        concatenation { '|' concatenation }
    concatenation:
        { closure }
    closure:
        term [ '*' | '+' | '?' ]
    term:
        '^'
        '$'
        '.'
        character
        '[' [ '^' ] character-ranges ']'
        '(' regexp ')'
*/
package regexp
```
关于注释书写，可参考 Go [官方文档说明](https://golang.org/doc/effective_go.html#commentary)

### 包内命名

在包内定义方法，类型时，外部必须使用包名调用。这样，你在给方法命名就不需要在包含包名了。如你不应将方法命名为`ChubbyFile`，别人调用时会变成`chubby.ChubbyFile`。实际上，你应命名为`File`，调用为`chubby.File`。

更多包内命名规范，可参考 Go [官方文档说明](http://golang.org/doc/effective_go.html#package-names)。

### 值传递
不要为了节省少量的内存空间，而将值类型数据通过指针方式传递给方法。值类型数据本身是固定大小的，不会太大，是可以直接传递的。当然这适合 struct 类型数据，不管多大多小。

### 对象方法
在定义对象方法时，对象名应该对象类型简称，一两个单词表示即可。如：
```go
func (cl *Client) Say(){...}
```
不要使用面向对象语言中常来指向自己的特定词，`this`，`me`，`self`等。这个命名不是描述方法的一部分，它的作用非常明显，该类型下所有方法定义时都应该使用一致的简称。不要在一个方法中定义为`cl`，而在其他方法中定义为`c`。在 Go 中应该保持代码的一致和精简。

### 入参类型
常常非常困惑方法入参是定义传值还是传指针，尤其是对新手。如果困惑，那么你就传指针，但有时候传值也是有道理的。总得判断标准是看执行效率，如大小不变的固定结构和基础类型数据，一些经验规则：

+ 如果是 map，func，chan 则不需要用指针，本身即是引用类型。
+ 如果是 slice ,而方法内部又不需要修改它们，则不需要用指针。
+ 如果是 func ，也需要方法内部替换，则需要使用指针
+ 如果是包含 sysnc.Mutex 的 struct ，则需要使用指针，避免对象复制。
+ 如果是一个大的 struct 或者 array ，则使用指针更加高效。到底多大才是大呢？粗暴点就是比方法对象还大。
+ 更多...

### 准确反馈测试失败信息
在 Testing 中，测试失败打印的错误信息应该包含输入的是什么，期望的是什么，得到的又是什么。也许会促使你写一个帮助类，但不管如何，你都需要让错误信息是有用的。即是别人来调试使用你的程序，也能明白意思。一个测试失败的示例：
```Go
if got != tt.want {
	//or Fatalf, if test can't test anything more past this point
    t.Errorf("Foo(%q) = %d; want %d", tt.in, got, tt.want)    
}
```
注意这里的顺序是**实际**!=**预期**，是消息提示的顺序，而 Go 中不需要想其他语言鼓励写出反向的。`0 != x`或者`expected 0, got x`。

这里提供一个 Go 官方的具有代表性的测试代码，源自 [fmt](http://golang.org/pkg/fmt/) 包。
```Go
var flagtests = []struct {
    in  string
    out string
}{
    {"%a", "[%a]"},
    {"%-a", "[%-a]"},
    {"%+a", "[%+a]"},
    {"%#a", "[%#a]"},
    {"% a", "[% a]"},
    {"%0a", "[%0a]"},
    {"%1.2a", "[%1.2a]"},
    {"%-1.2a", "[%-1.2a]"},
    {"%+1.2a", "[%+1.2a]"},
    {"%-+1.2a", "[%+-1.2a]"},
    {"%-+1.2abc", "[%+-1.2a]bc"},
    {"%-1.2abc", "[%-1.2a]bc"},
}

func TestFlagParser(t *testing.T) {
    var flagprinter flagPrinter
    for _, tt := range flagtests {
        s := Sprintf(tt.in, &flagprinter)
        if s != tt.out {
            t.Errorf("Sprintf(%q, &flagprinter) => %q, want %q", tt.in, s, tt.out)
        }
    }
}
```
上述示例中，在测试不通过时，能非常清晰的知道哪个方法测试不通过。`t.Errorf`不是断言，而是提示错误信息，并继续运行。

### 变量命名

变量命名以精简为佳，但也有具有描述性。特别是局部变量可以使用更短的命名，用**c**比 lineCount 好，用**i**比 sliceIndex 好。 

基本规则：命名必须具有描述性。对于对象方法，对象名用前面一两个单词做简称即可，优先使用一些常见的惯用词**i**，但全局性变量和不常见信息还是需要使用更多描述性内容的名称。

这篇文章能用来4小时断断续续才写完，当然是根据 Go 官方 [Wiki-CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments) 所写。对于 Go 编码规范和开发习惯具有非常高的参考价值，经过验证提炼的指导意见，能帮助我们写出更优美更极致的 Go 代码。 

[goimport]: https://github.com/bradfitz/goimports
