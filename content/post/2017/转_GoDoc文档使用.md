
---
date: 2017-03-13T08:20:09+08:00
title: "GoDoc文档使用"
description: ""
disqus_identifier: 1489364409199950008
slug: "Goyu-yan-shi-zhan-bi-ji-(san-)|-Go-Doc-wen-dang"
source: "http://www.jianshu.com/p/ed6a6d2f1fdb"
tags: 
- golang 
categories:
- 编程语言与开发
---

> 《Go语言实战》读书笔记，未完待续，欢迎关注公众号`flysnow_org`，第一时间看后续笔记。

对于协作开发或者代码共享来说，文档是一个可以帮助开发者快速了解以及使用这些代码的一个教程，文档越全面，越详细，入门越快，效率也会更高。

在Go语言中，Go为我们提供了快速生成文档以及查看文档的工具，让我们可以很容易的编写查看文档。

Go提供了两种查看文档的方式，一种是使用`go doc`命令在终端查看，这种适用于使用VIM等工具在终端开发的人员，它们不用离开终端，既可以查看想查看的文档，又可以编码。

第二种方式，是使用浏览器查看的方式，通过`godoc`命令可以在本机启动一个web服务，我们可以通过打开浏览器，访问这个服务来查看我们的Go文档。

从终端查看文档
--------------

这种方式适用于在终端开发的，它们一般不像离开终端，查完即可继续编码，这时候使用`go doc`命令是很不错的选择。

    ➜  hello go help doc
    usage: go doc [-u] [-c] [package|[package.]symbol[.method]]

    Doc prints the documentation comments associated with the item identified by its
    arguments (a package, const, func, type, var, or method) followed by a one-line
    summary of each of the first-level items "under" that item (package-level
    declarations for a package, methods for a type, etc.).

    Flags:
        -c
            Respect case when matching symbols.
        -cmd
            Treat a command (package main) like a regular package.
            Otherwise package main's exported symbols are hidden
            when showing the package's top-level documentation.
        -u
            Show documentation for unexported as well as exported
            symbols and methods.

从以上可以看出，`go doc`的使用比较简单，接收的参数是包名，或者以包里的结构体、方法等。如果我们不输入任何参数，那么显示的是当前目录的文档，下面看个例子。

    /*
     提供的常用库，有一些常用的方法，方便使用
     */
    package lib

    // 一个加法实现
    // 返回a+b的值
    func Add(a,b int) int {
        return a+b
    }

    ➜  lib go doc
    package lib // import "flysnow.org/hello/lib"

    提供的常用库，有一些常用的方法，方便使用

    func Add(a, b int) int

在当前目录执行`go doc`，输出了当前目录下的文档信息。

除此之外，我们还可以指定一个包，就可以列出当前这个包的信息，着包括文档、方法、结构体等。

    ➜  lib go doc json
    package json // import "encoding/json"

    Package json implements encoding and decoding of JSON as defined in RFC
    4627. The mapping between JSON and Go values is described in the
    documentation for the Marshal and Unmarshal functions.

    See "JSON and Go" for an introduction to this package:
    https://golang.org/doc/articles/json_and_go.html

    func Compact(dst *bytes.Buffer, src []byte) error
    func HTMLEscape(dst *bytes.Buffer, src []byte)
    func Indent(dst *bytes.Buffer, src []byte, prefix, indent string) error
    func Marshal(v interface{}) ([]byte, error)
    func MarshalIndent(v interface{}, prefix, indent string) ([]byte, error)
    func Unmarshal(data []byte, v interface{}) error
    type Decoder struct{ ... }
        func NewDecoder(r io.Reader) *Decoder
    type Delim rune
    type Encoder struct{ ... }
        func NewEncoder(w io.Writer) *Encoder
    type InvalidUTF8Error struct{ ... }
    type InvalidUnmarshalError struct{ ... }
    type Marshaler interface{ ... }
    type MarshalerError struct{ ... }
    type Number string
    type RawMessage []byte
    type SyntaxError struct{ ... }
    type Token interface{}
    type UnmarshalFieldError struct{ ... }
    type UnmarshalTypeError struct{ ... }
    type Unmarshaler interface{ ... }
    type UnsupportedTypeError struct{ ... }
    type UnsupportedValueError struct{ ... }

以上是我们以`json`包为例，查看该包的文档，从中我们可以看到它有一个名为`Decoder`的结构体，我们进一步查看这个结构体的文档。

    ➜  lib go doc json.Decoder
    package json // import "encoding/json"

    type Decoder struct {
        // Has unexported fields.
    }
        A Decoder reads and decodes JSON values from an input stream.


    func NewDecoder(r io.Reader) *Decoder
    func (dec *Decoder) Buffered() io.Reader
    func (dec *Decoder) Decode(v interface{}) error
    func (dec *Decoder) More() bool
    func (dec *Decoder) Token() (Token, error)
    func (dec *Decoder) UseNumber()

现在我们看到这个`Decoder`有很多方法,进一步查看这些方法的文档，比如`Decode`。

    ➜  lib go doc json.Decoder.Decode    
    func (dec *Decoder) Decode(v interface{}) error
        Decode reads the next JSON-encoded value from its input and stores it in the
        value pointed to by v.

        See the documentation for Unmarshal for details about the conversion of JSON
        into a Go value.

`go doc`使用就是这样，一步步，缩小范围，查看想看的那些包、结构体、接口或者函数方法的文档。

在线浏览文档
------------

`go doc`终端查看的方式，虽然也很便捷，不过效率不高，并且没有查看细节以及进行跳转，为此Go为我们提供了基于浏览器使用的网页方式进行浏览API
文档，我们只用点点鼠标，就可以查看了，还可以在方法、包等之间进行跳转，更简洁方便。

要想启动一个Web在线API文档服务很简单，使用`godoc`就可以了。

    ➜  lib godoc -http=:6060

后面的http是要指定Web服务监听的IP和Port，运行后，我们就可以打开浏览器，输入`http://127.0.0.1:6060`进行访问了，你会发现打开的页面，和GoLang的官方网站一样，没错，这个其实就是官网的一个拷贝，但是包的文档`http://127.0.0.1:6060/pkg/`会和官网不一样,你自己启动的这个服务，是基于你电脑上`GOROOT`和`GOPATH`这两个路径下的所有包生成的文档，会比官网只是标准库的文档要多。

在线浏览API文档非常方便，只需要鼠标点击就可以了，也可以点击蓝色的超链接在方法、结构、接口以及包等之间跳转，还可以查看对应的源代码，示例代码，很方便，我们经常用的也是这个在线浏览方式。

生成自己的文档
--------------

Go文档工具，还有一个亮点，就是可以支持开发人员自己写的代码，只要开发者按照一定的规则，就可以自动生成文档了。

在我们编码中，文档就是注释，Go语言采用了和C、Java差不多的注释风格。一种是双斜线的方式，一种是斜线和星号的方式。

    /*
     提供的常用库，有一些常用的方法，方便使用
     */
    package lib

    // 一个加法实现
    // 返回a+b的值
    func Add(a,b int) int {
        return a+b
    }

这还是我们刚刚那个例子，例子中文档的编写的两种风格。想要为哪些标识符生车文档，就在哪些标识符之前，使用注释的方式，加入到代码中即可。

现在我们不管是用`go doc`，还是`godoc`都可以看到我们刚刚注释的文档了。

添加文档示例
------------

我们在看很多官方API文档的时候，可以在文档里看到一些例子，这些例子会告诉我们怎么使用API，以及这个例子打印的输出是什么，我觉得这个非常好，这样看函数文档看不懂的，可以参考这个例子，那么对于我们自己写的API，怎么给API文档添加示例代码呢？

这里我参考了官方的源代码，总结了测试了一下，发现可行，这里分享一下。

1.  示例代码必须单独存放在一个文件中，文件名字为`example_test.go`。
2.  在这个go文件里，定义一个名字为`Example`的函数，参数为空
3.  示例的输出采用注视的方式，以//Output:开头，另起一行，每行输出占一行。

说了这三个规则，下面通过一个例子更直观的了解。

    package lib

    import "fmt"

    func Example() {
        sum:=Add(1,2)
        fmt.Println("1+2=",sum)
        //Output:
        //1+2=3
    }

这就是为刚刚那个`Add`函数写的示例代码，我们运行`godoc`就可以看到结果了。

![](https://static.yushuangqi.com/blog/2017/0313075858bpnsb0rg3be.png)\

示例代码文档

Go的文档工具非常强大，更多功能，我们可以使用帮助命令查看。这里再推荐一个比较不错的第三方的API文档网站，收录了包括官方在内的很多Go库，可以直接跳转，关联源代码，非常方便。[https://gowalker.org/](https://gowalker.org/)

《Go语言实战》读书笔记，未完待续，**欢迎关注公众号`flysnow_org`，第一时间看后续笔记。**

