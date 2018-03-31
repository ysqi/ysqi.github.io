
---
date: 2016-12-31T11:34:45+08:00
title: "Goforgophers"
description: ""
disqus_identifier: 1485833685500972071
slug: "Go-for-gophers"
source: "https://segmentfault.com/a/1190000000655746"
tags: 
- channels 
- concurrency 
- goroutine 
- gopher 
- golang 
categories:
- 编程语言与开发
---

> 注：该文是作者 Andrew Gerrand 在 GopherCon closing keynote\
> 25 April 2014 上的演讲，原文地址为 [Go for
> gophers](http://talks.golang.org/2014/go4gophers.slide#1)

> 注：这个是视频集合 [Watch the talk on
> YouTube](https://www.youtube.com/watch?v=dKGmK_Z1Zl0)，赞伟大的长城，需要翻墙INGINGING.

Interfaces
==========

Interfaces: 第一印象
--------------------

我曾经对 classes 和 types 感兴趣。

Go 反对这些：

-   没有继承
-   没有子类型多态
-   没有泛型

它反而强调 interfaces。

Interfaces: Go 的方式
---------------------

Go interfaces 是小的。

    type Stringer interface {
        String() string
    }

Stringer 能完美的打印它自己。\
任何实现了 String 的都是一个 Stringer。

一个 interface 示例
-------------------

一个 io.Reader 的值发出了一个二进制的数据流。

    type Reader interface {
        Read([]byte) (int, error)
    }

像一个 UNIX 管道。

实现 interfaces
---------------

    // ByteReader implements an io.Reader that emits a stream of its byte value.
    type ByteReader byte

    func (b ByteReader) Read(buf []byte) (int, error) {
        for i := range buf {
            buf[i] = byte(b)
        }
        return len(buf), nil
    }

封装 interfaces
---------------

    type LogReader struct {
        io.Reader
    }

    func (r LogReader) Read(b []byte) (int, error) {
        n, err := r.Reader.Read(b)
        log.Printf("read %d bytes, error: %v", n, err)
        return n, err
    }

使用一个 LogReader 封装一个 ByteReader

    r := LogReader{ByteReader('A')}
    b := make([]byte, 10)
    r.Read(b)
    fmt.Printf("b: %q", b)

通过封装我们构成了 interface 的值。

Chaining interfaces
-------------------

封装 wrappers 来构建 chains：

    var r io.Reader = ByteReader('A')
    r = io.LimitReader(r, 1e6)
    r = LogReader{r}
    io.Copy(ioutil.Discard, r)

更简洁：

    io.Copy(ioutil.Discard, LogReader{io.LimitReader(ByteReader('A'), 1e6)})

通过组合小的片段来实现复杂的行为。

使用 interfaces 编程
--------------------

Interfaces 从行为上分离数据。

interfaces, functions 能从表现上区分：

    // Copy copies from src to dst until either EOF is reached
    // on src or an error occurs.  It returns the number of bytes
    // copied and the first error encountered while copying, if any.
    func Copy(dst Writer, src Reader) (written int64, err error) {

     io.Copy(ioutil.Discard, LogReader{io.LimitReader(ByteReader('A'), 1e6)})

Copy 不知道底层数据结构。

一个更大的 interface
--------------------

sort.Interface 描述了要求排序一个 collection 的操作。

    type Interface interface {
        Len() int
        Less(i, j int) bool
        Swap(i, j int)
    }

IntSlice 可以排序一个 ints 的 slice ：

    type IntSlice []int

    func (p IntSlice) Len() int           { return len(p) }
    func (p IntSlice) Less(i, j int) bool { return p[i] < p[j] }
    func (p IntSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

sort.Sort 可以使用 IntSlice 排序一个 \[\]int：

    s := []int{7, 5, 3, 11, 2}
    sort.Sort(IntSlice(s))
    fmt.Println(s)

另外一个 interface 示例
-----------------------

Organ 类型描述了一个 body 部分以及它可以打印自己。

    type Organ struct {
        Name   string
        Weight Grams
    }

    func (o *Organ) String() string { return fmt.Sprintf("%v (%v)", o.Name, o.Weight) }

    type Grams int

    func (g Grams) String() string { return fmt.Sprintf("%dg", int(g)) }

    func main() {
        s := []*Organ{{"brain", 1340}, {"heart", 290},
            {"liver", 1494}, {"pancreas", 131}, {"spleen", 162}}

        for _, o := range s {
            fmt.Println(o)
        }
    }

排序 organs
-----------

Organs 类型怎样描述和改变一个 organs slice。

    type Organs []*Organ

    func (s Organs) Len() int      { return len(s) }
    func (s Organs) Swap(i, j int) { s[i], s[j] = s[j], s[i] }

ByName 和 ByWeight 类型通过不同的属性嵌入 Organs 来排序。

    type ByName struct{ Organs }

    func (s ByName) Less(i, j int) bool { return s.Organs[i].Name < s.Organs[j].Name }

    type ByWeight struct{ Organs }

    func (s ByWeight) Less(i, j int) bool { return s.Organs[i].Weight < s.Organs[j].Weight }

通过嵌入我们组合了类型。

为了排序 \[\]\*Organ，使用 ByName 或是 ByWeight 封装它，然后把它传给
sort.Sort：

        s := []*Organ{
            {"brain", 1340},
            {"heart", 290},
            {"liver", 1494},
            {"pancreas", 131},
            {"spleen", 162},
        }

        sort.Sort(ByWeight{s})
        printOrgans("Organs by weight", s)

        sort.Sort(ByName{s})
        printOrgans("Organs by name", s)

另外一个封装
------------

Reverse 函数获取了一个 sort.Interface 和 使用一个 inverted Less
方法返回一个 sort.Interface：

    func Reverse(data sort.Interface) sort.Interface {
        return &reverse{data}
    }

    type reverse struct{ sort.Interface }

    func (r reverse) Less(i, j int) bool {
        return r.Interface.Less(j, i)
    }

为了使用降序排序 organs，使用 Reverse 组合我们的 sort 类型。

        sort.Sort(Reverse(ByWeight{s}))
        printOrgans("Organs by weight (descending)", s)

        sort.Sort(Reverse(ByName{s}))
        printOrgans("Organs by name (descending)", s)

Interfaces: 为什么这样做
------------------------

他们不仅仅是非常 cool 的技巧。\
这是我们如何在 Go 中结构化编程。

Interfaces: Sigourney
---------------------

Sigourney 是一个我用 Go 编写的模块化的音频合成器。

音频是由一连串的 Processor 生成。

    type Processor interface {
        Process(buffer []Sample)
    }

（[github.com/nf/sigourney](https://github.com/nf/sigourney)）

Interfaces: Roshi
-----------------

Roshi 是一个 Peter Bourgon 编写的时间序列事件存储，它提供 API：

    Insert(key, timestamp, value)
    Delete(key, timestamp, value)
    Select(key, offset, limit) []TimestampValue

同样的 API 是由系统的 farm 和 cluster 部分实现：

展示组合的一个优雅设计：

([github.com/soundcloud/roshi](https://github.com/soundcloud/roshi))

Interfaces: 为什么这样做
------------------------

Interfaces 是泛型编程机制。\
他们给了 Go 一个熟悉的形式。\
少即是多。

这都是组成。\
Interfaces - 通过设计和规范 - 鼓励我们编写可组合的代码。

Interfaces 类型仅仅是类型。\
interface 值仅仅是值。\
对于其他语言，它们是正交的。

Interfaces 从行为区分数据。（Classes 合并它们）。

    type HandlerFunc func(ResponseWriter, *Request)

    func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
        f(w, r)
    }

Interfaces: 我学到了什么
------------------------

多思考组合。\
做很多小的事情比做一个大而复杂的事情更好。\
并且：我认为小也是相当大的。\
当大是有益的，一些重复的小也是好的。

Concurrency
===========

Concurrency: 第一印象
---------------------

我第一次接触并发是在：C, Java, 和 Python 中。\
然后：在 Python 和 JavaScript 接触事件驱动模型。

当我看到 Go 时，我看到的是：

“一个没有回调的高效事件驱动模型”。

但是我还有问题：“为什么我不能等待或是 kill 一个 goroutine？”

Concurrency: Go 的方式
----------------------

Goroutines 提供并发执行。

Channels 表示通讯和同步是用独立的进程。

Select 使得在 channel 操作上运算。

一个并发示例
------------

来自于 Go Tour 的二叉树的比较执行。

“实现一个函数

    func Same(t1, t2 *tree.Tree) bool

来比较两个二叉树的内容”

Walking a tree
--------------

    type Tree struct {
        Left, Right *Tree
        Value int
    }

一个简单的深度优先树的遍历：

    func Walk(t *tree.Tree) {
        if t.Left != nil {
            Walk(t.Left)
        }
        fmt.Println(t.Value)
        if t.Right != nil {
            Walk(t.Right)
        }
    }

    func main() {
        Walk(tree.New(1))
    }

一个并发的 walker：

    func Walk(root *tree.Tree) chan int {
        ch := make(chan int)
        go func() {
            walk(root, ch)
            close(ch)
        }()
        return ch
    }

    func walk(t *tree.Tree, ch chan int) {
        if t.Left != nil {
            walk(t.Left, ch)
        }
        ch <- t.Value
        if t.Right != nil {
            walk(t.Right, ch)
        }
    }

并发的 Walking 两个树：

    func Same(t1, t2 *tree.Tree) bool {
        w1, w2 := Walk(t1), Walk(t2)
        for {
            v1, ok1 := <-w1
            v2, ok2 := <-w2
            if v1 != v2 || ok1 != ok2 {
                return false
            }
            if !ok1 {
                return true
            }
        }
    }

    func main() {
        fmt.Println(Same(tree.New(3), tree.New(3)))
        fmt.Println(Same(tree.New(1), tree.New(2)))
    }

不使用 channels 比较树
----------------------

    func Same(t1, t2 *tree.Tree) bool {
        w1, w2 := Walk(t1), Walk(t2)
        for {
            v1, ok1 := w1.Next()
            v2, ok2 := w2.Next()
            if v1 != v2 || ok1 != ok2 {
                return false
            }
            if !ok1 {
                return true
            }
        }
    }

Walk 函数几乎有相同的签名：

    func Walk(root *tree.Tree) *Walker {
    func (w *Walker) Next() (int, bool) {

（我可以调用 Next 代替 channel receive）

但是实现是更加复杂的：

    func Walk(root *tree.Tree) *Walker {
        return &Walker{stack: []*frame{{t: root}}}
    }

    type Walker struct {
        stack []*frame
    }

    type frame struct {
        t  *tree.Tree
        pc int
    }

    func (w *Walker) Next() (int, bool) {
        if len(w.stack) == 0 {
            return 0, false
        }

        // continued next slide ...
        f := w.stack[len(w.stack)-1]
        if f.pc == 0 {
            f.pc++
            if l := f.t.Left; l != nil {
                w.stack = append(w.stack, &frame{t: l})
                return w.Next()
            }
        }
        if f.pc == 1 {
            f.pc++
            return f.t.Value, true
        }
        if f.pc == 2 {
            f.pc++
            if r := f.t.Right; r != nil {
                w.stack = append(w.stack, &frame{t: r})
                return w.Next()
            }
        }
        w.stack = w.stack[:len(w.stack)-1]
        return w.Next()
    }

另一个 channel 版本
-------------------

    func Walk(root *tree.Tree) chan int {
        ch := make(chan int)
        go func() {
            walk(root, ch)
            close(ch)
        }()
        return ch
    }

    func walk(t *tree.Tree, ch chan int) {
        if t.Left != nil {
            walk(t.Left, ch)
        }
        ch <- t.Value
        if t.Right != nil {
            walk(t.Right, ch)
        }
    }

但是有一个问题：当 inequality 被发现，一个 goroutine 发送给 ch
可能会被阻塞。

Stopping early
--------------

给 walker 加入一个 quit channel 以便我们可以停止它。

    func Walk(root *tree.Tree, quit chan struct{}) chan int {
        ch := make(chan int)
        go func() {
            walk(root, ch, quit)
            close(ch)
        }()
        return ch
    }

    func walk(t *tree.Tree, ch chan int, quit chan struct{}) {
        if t.Left != nil {
            walk(t.Left, ch, quit)
        }
        select {
        case ch <- t.Value:
        case <-quit:
            return
        }
        if t.Right != nil {
            walk(t.Right, ch, quit)
        }
    }

创建一个 quit channel 并传给每个 walker。\
当 Same 退出的时候，通过关闭 quit，任何正在运行的 walkers 都将中断。

    func Same(t1, t2 *tree.Tree) bool {
        quit := make(chan struct{})
        defer close(quit)
        w1, w2 := Walk(t1, quit), Walk(t2, quit)
        for {
            v1, ok1 := <-w1
            v2, ok2 := <-w2
            if v1 != v2 || ok1 != ok2 {
                return false
            }
            if !ok1 {
                return true
            }
        }
    }

为什么不仅仅 kill goroutines？
------------------------------

Goroutines 在 Go 的代码中是不可见的。不能杀掉它或是等待。

你已经自己构建了。

这里是原因：\
一旦 Go 代码知道它运行的哪个 thread，你就能你得到 thread-locality 。\
Thread-locality 使得并发模型失败。

Concurrency: why it works
-------------------------

这个模型使得 concurrent 代码可读和可写。\
（使得并发是可理解的）

鼓励分解独立的计算。

简单的并发模型使得它足够灵活。\
Channels 仅仅是值，它们适合正确的类型系统。

Goroutines 在 Go 代码中是不可见的，这可以让你在任何地方 concurrency 。

少即是多。

Concurrency: 我学到了什么
-------------------------

Concurrency 不仅仅是做更快的做更多事情。\
编写更好的代码。

语法
====

Syntax: 第一印象
----------------

首先， Go 的语法一点也不刻板和冗长。\
我习惯了它提供的便利。

例如：

-   在属性中没有 getters/setters
-   没有 map/filter/reduce/zip
-   没有可选参数

Syntax: Go 的方式
-----------------

可读性优于一切。\
提供足够的语法糖使得它有效率，但是不会太多。

Getters and setters (or "properties")
-------------------------------------

Getters and setters 使得 assignments 和 reads 变成函数调用。\
这会导致令人惊讶的隐藏行为。

在 Go 中，仅仅 write (and call) 方法。

控制流不会被掩盖。

Map/filter/reduce/zip
---------------------

Map/filter/reduce/zip 在 Python 中非常有用：

    a = [1, 2, 3, 4]
    b = map(lambda x: x+1, a)

在 Go 中，你只能写循环。

    a := []int{1, 2, 3, 4}
    b := make([]int, len(a))
    for i, x := range a {
        b[i] = x+1
    }

这有一点冗长。\
但是使得性能特性更明显。

很容易写代码，并且你可以得到更加多的掌控。

可选参数
--------

Go 的函数没有可选参数。

使用函数变化代替：

    func NewWriter(w io.Writer) *Writer
    func NewWriterLevel(w io.Writer, level int) (*Writer, error)

或是使用一个 options struct：

    func New(o *Options) (*Jar, error)

    type Options struct {
        PublicSuffixList PublicSuffixList
    }

或是一个可变的选项列表。

创建小而简单的事情，而不是大而复杂的事情。

Syntax: why it works
--------------------

该语言拒绝复杂的代码。

使用明显的控制流，可以非常容易的进入不熟悉的代码。

相反，我们创建更加的事情，使得非常容易记录文档和明白。

因此 Go 代码非常容易读。

（使用 gofmt，会使得代码更加可读）

Syntax: 我学到了什么
--------------------

我是非常聪明的为自己好。

我非常欣赏 Go 代码的一致性，清晰性和透明性。

我有时候会丢失便利性，但是很少。

错误处理
========

错误处理：第一印象
------------------

我以前使用 exceptions 处理过错误。

通过比较，Go 的错误处理模型非常冗长。

我是立即讨厌键入这个：

    if err != nil {
        return err
    }

Error handling: Go 的方式
-------------------------

Go 使用内建的内建的 error 接口编码错误：

    type error interface {
        Error() string
    }

Error 的值使用起来就像其他任何值。

    func doSomething() error

    err := doSomething()
    if err != nil {
        log.Println("An error occurred:", err)
    }

错误处理的代码仅仅是代码。

（以一个约定（os.Error）开始），在 Go 1 是内建的。

Error handling: why it works
----------------------------

错误处理被引进。

Go 使得错误处理和其他任何代码一样重要。

Errors 仅仅是值，它们很容易融入语言的其他部分（interfaces, channels
等等）。

结果：Go 代码处理错误是正确的且优雅的。

我们为错误使用同样的语言。\
没有隐藏的控制流（throw/try/catch/finally）提升了可读性。

少即是多。

Error handling: 我学到了什么
----------------------------

为了写出更好的代码，必须考虑错误处理。

Exceptions 使得非常容易避免思考 errors。\
（错误不应该是异常）

Go 鼓励我们考虑每一种错误情况。

我的 Go 程序比我的其他程序更具有鲁棒性。\
（我根本不会错过错误。）

Packages
========

Packages: 第一印象
------------------

我发现 capital-letter-visibility 规则很怪异；\
“让我使用我自己的命名方案！”

我不喜欢每个目录一个包；\
“让我使用我自己的结构！”

我对于缺乏 monkey patching 非常失望。

Packages: Go 的方式
-------------------

Go packages 是一个类型、函数、变量和常量的命名空间。

Visibility
----------

Visibility 在包级别。

当它们使用一个大写字母的时候，Names 被导出。

    package zip

    func NewReader(r io.ReaderAt, size int64) (*Reader, error) // exported

    type Reader struct {    // exported
        File    []*File     // exported
        Comment string      // exported
        r       io.ReaderAt // unexported
    }

    func (f *File) Open() (rc io.ReadCloser, err error)   // exported

    func (f *File) findBodyOffset() (int64, error)        // unexported

    func readDirectoryHeader(f *File, r io.Reader) error  // unexported

好的可读性：非常容易的知道一个名字是否是公共接口的一部分\
好的设计：couples naming decisions with interface decisions

Package 结构
------------

Packages 可以跨越多个文件传播。

允许共享私有的实现和非正式的代码组织。

Packages 文件必须存在在包的唯一目录中。

目录的路径绝对了 import 的路径。

构建系统查找依赖从源码中独立。

"Monkey patching"
-----------------

GO 禁止从包外面修改包的声明。

但是我们可以使用全局变量实现类似的行为：

    package flag

    var Usage = func() {
        fmt.Fprintf(os.Stderr, "Usage of %s:\n", os.Args[0])
        PrintDefaults()
    }

或是注册函数：

    package http

    func Handle(pattern string, handler Handler)

这给了 monkey patching 足够的灵活性但是要注意包上作者的条款。

（这依赖于 Go 的初始化语义）。

Packages: why they work
-----------------------

松散的包组织让我们写代码和重构代码都很容易。

但是包鼓励程序员考虑公共接口。

这导致了好的命名和简单的接口。

源作为唯一的值得信赖的来源，它们没有 makefiles 来同步。

（这个涉及促使了好的工具如 [godoc.org](http://godoc.org/) 和
goimports）。

可预测的语义使得包非常容易读，明白以及使用。

Packages: 我学到了什么
----------------------

Go 的包教会了我优先考虑我的代码的使用者。\
（即使这使用者是我）

它也阻止了我做恶心的东西。

在任何情况下，包都是精确的。\
那种感觉还不错。

也许是我最喜欢的语言的一部分。

Documentation
=============

Documentation: 第一印象
-----------------------

Godoc 从 Go 的源码读文档，像 pydoc 或 javadoc。

但是与这两个不同的是，它不支持复杂的格式或者是其他的元数据。

为什么？

Documentation: Go 的方式
------------------------

Godoc 注释在一个导出的声明标示符之前：

    // Join concatenates the elements of a to create a single string.
    // The separator string sep is placed between elements in the resulting string.
    func Join(a []string, sep string) string {

它提取注释并且显示它们：

    $ godoc strings Join
    func Join(a []string, sep string) string
        Join concatenates the elements of a to create a single string. The
        separator string sep is placed between elements in the resulting string.

也集成测试框架来提供测试函数示例：

    func ExampleJoin() {
        s := []string{"foo", "bar", "baz"}
        fmt.Println(strings.Join(s, ", "))
        // Output: foo, bar, baz
    }

Documentation: why it works
---------------------------

Godoc 想让你写更好的注释，因此这源码看起来不错：

    // ValidMove reports whether the specified move is valid.
    func ValidMove(from, to Position) bool

Javadoc 仅仅想生成漂亮的文档，因此源码看起来是丑陋的。

    /**
     * Validates a chess move.
     *
     * @param fromPos  position from which a piece is being moved
     * @param toPos    position to which a piece is being moved
     * @return         true if the move is valid, otherwise false
     */
    boolean isValidMove(Position fromPos, Position toPos)

（一个 "ValidMove" 的 grep 会返回文档的第一行）

Documentation: 我学到了什么
---------------------------

Godoc 教会了我如写代码一样写文档。\
写文档提升了我写代码的技能。

更多
----

这里有许多示例。

最重要的主题：

-   首先，一些东西看起来奇怪或是缺乏。
-   我认识到那是一个设计决定。

这些决定使得这个语言 - 和 Go 代码 - 更好

有时候，你应该和一个语言生活一段时间再去看它。

经验教训
========

代码是用来交流的
----------------

说清楚：

-   选择一个好的名字
-   设计简单的接口
-   写精确的文档
-   不要自作聪明

少即是多
--------

新特性会减弱已经存在的特性。\
特性使复杂度增加。\
复杂性击败正交性。---- Complexity defeats orthogonality
（这个真的是这样翻译的吗？求大神）。\
正交性是至关重要的 - 它有利于组合。

组合是关键
----------

不要通过构建一个事情来解决问题。\
组合简单的工具并且组成它们来代替。

设计好的接口
------------

-   不要过分细化
-   寻找关键点（靶心）
-   不要太粗糙

简化是困难的
------------

花时间找出简单的解决方案。

Go 对我的影响
-------------

这些经验教训是我所知道的所有事情。

Go 帮助我认识到了它们。

Go 使得我变成了一个更好的程序员。

一个给任何地方的 gophers 的信息
-------------------------------

让我们一起构建小的，简单的，漂亮的东西。



