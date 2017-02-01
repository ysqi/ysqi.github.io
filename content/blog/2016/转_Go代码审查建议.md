
---
date: 2016-12-31T11:34:46+08:00
title: "Go代码审查建议"
description: ""
disqus_identifier: 1485833686279457789
slug: "Go-dai-ma-shen-cha-jian-yi"
source: "https://segmentfault.com/a/1190000000654529"
tags: 
- golang 
topics:
- 编程语言与开发
---

> 注：该文的原文来自于 go-wiki 为 [Go Code Review
> Comments](https://code.google.com/p/go-wiki/wiki/CodeReviewComments)

Go 代码审查建议
===============

该页收集了 Go
代码审查时候的常见意见，以至于一个详细说明能被快速参考。这是一个常见的错误清单，而不是一个风格指南。

你可以看 [effective go](http://golang.org/doc/effective_go.html)
作为补充。

**请在编辑这个页面前先讨论这个变更**，就算是一个很小的变更，许多人都有自己的想法，这里不是战场。

gofmt
-----

运行 gofmt
来自动化的解决你代码的主要的机械的风格问题，几乎所有的不正规的 go
代码都使用 gofmt。该文档的其余部分涉及非机械式的风格点。

另外一个替代方案是使用
[goimports](https://godoc.org/code.google.com/p/go.tools/cmd/goimports)，它是
gofmt 的父集，额外的新增（和移除）了 import 行。

注释句子
--------

查看
<http://golang.org/doc/effective_go.html#commentary>。注释文档声明了应该是整句，即使它看起来是有点多余的。这个方法使得它被提取进
godoc
文档的时候，是非常格式化的。注释应该开始于这个事物被描述的名字，并在一段时期结束。

    // A Request represents a request to run a command.
    type Request struct { ...

    // Encode writes the JSON encoding of req to w.
    func Encode(w io.Writer, req *Request) { ...

等等。

Doc 建议
--------

所有顶级的，导出的名字都应该有文档注释，作为 non-trivial unexported type
和功能声明。看 <http://golang.org/doc/effective_go.html#commentary>
来获取更多的注释约定的更多信息。

不要使用 Panic
--------------

看 <http://golang.org/doc/effective_go.html#errors>，不要使用 panic
用于正常的错误处理。使用 error 和多个返回值。

Error Strings
-------------

Error strings
不应该大写（除非有专有名词或是首字母缩小）或是以标点符号结束。因为它们通常在其他上下文中打印。即使用
fmt.Errorf("something bad") 而不是 fmt.Errorf("Something bad")，以至于
log.Print("Reading %s: %v", filename, err)
格式没有一个虚假的大写字母中间消息。这个不适用于日志记录，这个是绝对面向行的，不被包含在其他信息中。

> 注：上面 Errorf 区别是里面的内容 s 这个 字母一个大写一个小写】

处理错误
--------

看 <http://golang.org/doc/effective_go.html#errors>，不要使用 \_
变量丢弃错误，如果一个函数返回一个错误，检查它确保函数式成功的。处理这个错误，返回它，或是在真正的异常情况下使用
panic。

Imports
-------

Imports 是以组的形式组织的，在它们之间使用空行，标准包在第一组。

    package main

    import (
        "fmt"
        "hash/adler32"
        "os"

        "appengine/user"
        "appengine/foo"

        "code.google.com/p/x/y"
        "github.com/foo/bar"
    )

[goimports](https://godoc.org/code.google.com/p/go.tools/cmd/goimports)
将帮助你做到这个。

Import 的点号
-------------

import
的点号形式对测试是非常有用的，由于循环依赖，不能成为被测试的包的一部分。

    package foo_test

    import (
        . "foo"
        "bar/testutil"  // also imports "foo"
    )

在这个例子中，测试文件不能在 foo 包中，因为它使用 bar/testutil，它也引入
foo，因此我们使用 'import .' 形式来伪装成 foo
包的一部分，即使它不是。除了这种情况，不要在你的程序中使用 'import .'
。它会使得你的程序难以阅读，因为它是难以理解的，一个名字比如 Quux
在当前包种是否是一个顶级的标识符或是在一个引入包中。

Indent Error Flow
-----------------

在一个最小的代码缩进中尝试保存正常的代码路径，和缩进错误处理，首先处理错误。这样做提升了程序的可读性，这个符合视觉的快速扫描习惯。例如，不要这些写：

    if err != nil {
        // error handling
    } else {
        // normal code
    }

而应该这样写：

    if err != nil {
        // error handling
        return // or continue, etc.
    }
    // normal code

如果 if 语句有一个初始化的语句，像这样：

    if x, err := f(); err != nil {
      // error handling
      return
    } else {
      // use x
    }

这就要求把短的变量声明移动到它自己的行去。

    x, err := f()
    if err != nil {
      // error handling
      return
    }
    // use x

首字母缩写
----------

名字的单词应该是首字母缩写的（比如，"URL" 或
"NATO"）是一致的情况。例如，"URL" 应该以 "URL" 或是 ”url“ 展现（就像
"urlPony", 或 "URLPony"），绝不是 Url 。这里有一个示例：ServeHTTP 而不是
ServeHttp。

这个规则也适用于 "ID"，当它是 "identifier" 短名称的时候。因此使用
"appID" 代替 "appId"。

被 protocol buffer
编译器生成的代码是脱离了这个规则的。人类写的代码应该比机器写的代码更高标准。

行长度
------

这在 Go
的代码中没有严格的限制，但是为了避免太长的行，同样的，当他们在更可读的长度的时候，不应该添加换行符使其更短
--- 比如，如果他们是重复的。

建议是通常在包装前不超过 80
个字符，不是因为它是一个规则，而是因为它在一个可显示几百列的编辑器中查看的时候可读性更高。人类更适合窄文本相对于一个宽文本。不管怎样，godoc
应该以更好的方式渲染它。

Mixed Caps
----------

看
<http://golang.org/doc/effective_go.html#mixed-caps>，这个甚至当它打破了其他语言的习惯的时候也适用。比如一个未导出的常量是
maxLength 而不是 MaxLength 或是 MAX\_LENGTH。

结果参数命名
------------

考虑下这个在 godoc 应该是看起来像什么，结果参数命名像：

    func (n *Node) Parent1() (node *Node)
    func (n *Node) Parent2() (node *Node, err error)

更好的用法是：

    func (n *Node) Parent1() *Node
    func (n *Node) Parent2() (*Node, error)

换句话说，如果一个函数返回了两个或是三个相同类型的参数，或者如果一个结果的含义通过上下文不清楚，添加命名会非常有用，比如：

    func (f *Foo) Location() (float64, float64, error)

没有这个清晰：

     // Location returns f's latitude and longitude.
     // Negative values mean south and west, respectively.
     func (f *Foo) Location() (lat, long float64, err error)

Naked
返回值是好的，如果函数很小。一旦它是一个中型函数，必须明确你的返回值，必然的结果：仅仅使得你使用
naked
返回值是不知道命名结果参数的。清晰的文档一直是比在你的函数中保存一行或两行更重要。

最后，在一些情况下，为了在一个 deferred closure
中改变它，你需要命名一个结果参数。这通常是没有问题的。

Naked Returns
-------------

查看
[CodeReviewComments\#Named\_Result\_Parameters](https://code.google.com/p/go-wiki/wiki/CodeReviewComments#Named_Result_Parameters)

包建议
------

Package 建议，就像所有的注释都应该由 godoc
呈现，必须与包相邻，而没有空行：

    // Package math provides basic constants and mathematical functions.
    package math

    /*
    Package template implements data-driven templates for generating textual
    output such as HTML.
    ....
    */
    package template

查看 <http://golang.org/doc/effective_go.html#commentary>
获取更多关于注释约定的信息。

包名
----

在你的包中的所有的引用的名称应该使用包名，因此你可以忽略这个名字的标识符。比如，如果你在包
chubby 中，你不需要键入 ChubbyFile，客户端将写成
chubby.ChubbyFile。相反，命名该类型文件，客户端将写成 chubby.File，看
<http://golang.org/doc/effective_go.html#package-names> 获取更多信息

传递值
------

不要将指针作为函数参数传递只是为了省几字节，如果函数引用的参数 x
仅仅至始至终都作为
*x，那么参数就不应该使用指针。常见的实例包括传递一个指针给一个 string
（*string），或是一个指针给一个 interface
值（\*io.Reader）。在两个情况中，它自己的值都是不变的，可以被直接传递。这个建议不适用于大的
structs，或是会增长的小的 structs。

Receiver Names
--------------

一个方法的 receiver
的名字应该是它身份的一个反射；通常一两个字母是足够满足它类型的缩写（比如，"c"
or "cl" 作为 "Client" 的缩写）。不要使用通用名称，比如 "me", "this" or
"self"，面向对象语言的典型标识是更注重方法而不是函数。该名字不必作为方法参数的描述，它的角色是非常明显的，并且不是作为文档目的。它可以是非常短的，因为它出现在每一个方法的几乎每一行上；熟悉承认整洁。需要保持一致，如果你在一个方法中调用
"c" receiver，不能在另外一个方法中调用 "cl"。

Receiver Type
-------------

在一个方法上选择是使用一个值 receiver 还是指针 receiver
是非常困难的，特别是对于 Go
新手。如果不能肯定，那就使用指针，但是有时候一个值 receiver
更有意义。通常是因为效率的原因，比如小的不变的 structs
或是基础类型的值。\
根据经验，一般来说：

-   如果 receiver 是一个 map，func 或 chan，不要使用指针
-   如果 receiver 是一个 slice 和 方法不再 reslice 或是 重分配的
    slice，不要使用指针
-   如果方法需要可变的 receiver，receiver 必须使用指针
-   如果 receiver 是一个包含 sync.Mutex 或是类似的 synchronizing 属性的
    struct，receiver 必须是指针以避免复制
-   如果 receiver 是一个大的 struct 或是 array，一个指针 receiver
    会更有效率。多大是大？假设它是要传递它所有的元素作为方法的参数。如果感到太大，它同样对于
    receiver 也是大的。
-   Can function or methods, either concurrently or when called from
    this method, be mutating the receiver?
    当方法被调用是，一个值类型的副本被创建，因此外部更新，不会影响
    receiver。如果在原始的 receiver 中改变必须是可见的，那 receiver
    必须是指针。
-   如果 receiver 是一个 struct, array 或 slice 和
    它任何一个元素是指针，那它即是可变的。更喜欢指针
    receiver，因为对于读者来说，它的意图更加清晰
-   如果 receiver 是小的 array 或 struct，那自然是值类型（比如，
    time.Time
    类型），没有可变的属性和没有指针，或者仅仅是一个简单的基础类型，比如
    int 或 string，一个值 receiver 会更有意义。一个值 receiver
    可以减少生成的内存垃圾数量；如果一个值被传递给一个值方法，一个栈拷贝将被使用，而不是在
    heap
    上分配内存（编译器尝试聪明的避免分配内存，但它可能不会一直成功）。在没有优化前，没有因为这个原因选择一个值
    receiver。
-   最后，当不确定的时候，请使用指针 receiver

Useful Test Failures
--------------------

测试应该在失败的时候伴随着输出有帮助的信息告诉你失败原因是什么，输入是什么，实际发生了什么，期望值是什么。它很可能成为写一堆
assertFoo
的帮手。但是确保你的帮手生成了有用的错误信息。假设一个不是你的人，或者不是你团队的人在
debugging 你的错误测试。一个典型的 Go 失败测试应该像这样：

            if got != tt.want {
                    t.Errorf("Foo(%q) = %d; want %d", tt.in, got, tt.want)    // or Fatalf, if test can't test anything more past this point
            }

注意这里的实际命令应该是期望 !=
，并且消息也使用这个命令。一些测试框架鼓励写这些后置的写法：0 != x，
“期望 x 获得 0”等等，Go 不这样做。

如果看起来有很多类型，你可能想写一个 table-driven
测试：<http://code.google.com/p/go-wiki/wiki/TableDrivenTests>。

另外一个常用的技术是消除错误测试的歧义，当使用一个有不同输入的来使用一个不同的
TestFoo 函数来包装每个调用测试帮手的时候，因此这个失败测试使用的名字为：

         func TestSingleValue(t *testing.T) { testHelper(t, []int{80}) }
         func TestNoValues(t *testing.T) { testHelper(t, []int{}) }

在任何情况下，你的责任是不管以后谁调试你的代码，当失败的时候，都应该输出有用的信息。

变量命名
--------

在 Go 中的变量名应该短而不是长。对于空间有限的局部变量尤其如此。更喜欢 c
表示行数，喜欢 i 表示 slice 的索引。

基本的规则是：进一步声明，一个名字被使用，这个名字必须描述更多的信息。对于一个方法
receiver，一个或两个字母是合适的。普通的变量比如 loop indices 和 readers
可以是单个字母（i, r）。更不寻常的事情和全局变量需要更具描述性的名称。

