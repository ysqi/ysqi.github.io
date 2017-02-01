
---
date: 2016-12-31T11:34:12+08:00
title: "Go语言快速入门"
description: ""
disqus_identifier: 1485833652557014296
slug: "Goyu-yan-kuai-su-ru-men"
source: "https://segmentfault.com/a/1190000003856830"
tags: 
- golang 
topics:
- 编程语言与开发
---

一年前为了给同事介绍Go而做的演讲文稿。一年过去，我对Go的理解没有任何进展。\
最近决定深入地学习这门语言。

Hello World
===========

    package main

    import "fmt"

    // this is a comment

    func main() {
        fmt.Println("Hello World")
    }

Build & Run
===========

    $ cd D:\Go\src\github.com\sif\hello

    $ go run main.go
    Hello World

    $ go build main.go

    $ main.exe
    Hello World

    $ go clean

Executable Size
===============

Language

Executable Size (KB)

Comments

Go

1524

C++

12

Using cout. \#include &lt;iostream&gt;

C++ (with runtime)

48

Using cout. \#include &lt;iostream&gt;

C

7

Using printf() \#include &lt;stdio.h&gt;

Go embeds the “virtual machine” into the executable.

Enviroment Variables
====================

On 64bit Windows:

  Name     Value     Comments
  -------- --------- ---------------------------------------------------------------
  GOARCH   amd64     
  GOOS     windows   
  GOPATH   D:\\Go    Location of your workspaces. Can specify multiple workspaces.
  GOROOT   C:\\Go    

**GOPATH**可以指定多个workspaces。

Workspace Hierarchy
===================

    bin\
        hello.exe

    pkg\
        windows_amd64\
            carestream.com\dental\
                csi.a

    src\
        github.com\sif\
            hello\
                hello.go
        carestream.com\dental\
            csi\
                patient.go

Doc Server
==========

Browse Go documents on <http://localhost>:6060/

Basic Types
===========

### Integer Numbers

*Integer types:*\
`uint8, uint16, uint32, uint64, int8, int16, int32, int64`

*Two alias types:*\
`byte (uint8) rune (int32)`

*Three machine dependent integer types:*\
`uint, int, uintptr`\
They are machine dependent because their size depends on the type of
architecture you are using.

*Floating Point Numbers*\
`float32, float64`\
Generally we should stick with float64 when working with floating point
numbers.

*Complex Numbers*\
`complex64, complex128`

Operators +, -, \*, / and % are all the same as C.

### Booleans

    func main() {
        fmt.Println(true && true)
        fmt.Println(true && false)
        fmt.Println(true || true)
        fmt.Println(true || false)
        fmt.Println(!true)
    }

### Strings

Go strings are made up of individual bytes, usually one for each
character.

    "Hello \nWorld"  // Similar to C/C++

    `Hello
     World`  // Similar to Python “””

    func main() {
        fmt.Println(len("Hello World"))
        fmt.Println("Hello World"[1])
        fmt.Println("Hello " + "World")
    }

Characters from other languages like Chinese are represented by more
than one byte.

    func main() {
        s := "abc汉字"

        for i := 0; i < len(s); i++ { // byte
            fmt.Printf("%c,", s[i])
        }

        fmt.Println()
        for _, r := range s { // rune
            fmt.Printf("%c,", r)
        }
    }

Output:

    a,b,c,æ,±,,å,,,
    a,b,c,汉,字,

### Enum

    // A Month specifies a month of the year (January = 1, ...).
    type Month int

    const (
        January Month = 1 + iota
        February
        March
        April
        May
        // ...
        November
        December
    )

Variables
=========

    func main() {
        var x string = "Hello World"
        fmt.Println(x)
    }

Shorter form:

    x := "Hello World"

The type is not necessary because the Go compiler is able to infer the
type based on the literal value you assign the variable.\
The compiler can also do inference with the var statement:

    var x = "Hello World"

Generally you should use this shorter form whenever possible.

### Scope

The same as C.

### Constants

Constants are basically variables whose values cannot be changed later.

    func main() {
        const x string = "Hello World"
        fmt.Println(x)
    }

    const x string = "Hello World"
    x = "Some other string"  // cannot assign to x

Constants must be numbers, strings and booleans which can be determined
in compile time.

### Defining Multiple Values

    var (
        a = 5
        b = 10
        c = 15
    )

    const (
        Sunday Weekday = iota
        Monday
        Tuesday
        Wednesday
        Thursday
        Friday
        Saturday
    )

Control Structures
==================

### IF

可省略条件表达式括号。\
支持初始化语句，可定义代码块局部变量。\
代码块左大括号必须在条件表达式尾部。

    x := 0

    if n := "abc"; x > 0 {
        fmt.Println(n[2])
    } else if x < 0 {
        fmt.Println(n[1])
    } else {
        fmt.Println(n[0])
    }

但是，不支持三元操作符 "a &gt; b ? a : b"。

### FOR

    s := "abc"

    for i, n := 0, len(s); i < n; i++ {    // 常见的 for 循环
        fmt.Println(s[i])
    }

    n := len(s)
    for n > 0 {                 // 替代 while (n > 0) {}
        fmt.Println(s[n])             // 替代 for (; n > 0;) {}
        n--
    }

    for {                         // 替代 while (true) {}
        fmt.Println(s)             // 替代 for (;;) {}
    }

### Range

类似迭代器操作，返回（索引，值）或（键，值）。

                1st value   2nd value    
  ------------- ----------- ------------ ---------------
  string        index       s\[index\]   unicode, rune
  array/slice   index       s\[index\]   
  map           key         m\[key\]     
  channel       element                  

    s := "abc"

    // 忽略 2nd value，支持 string/array/slice/map。
    for i := range s {
        fmt.Println(s[i])
    }

    for _, c := range s {     // 忽略 index
        fmt.Println(c)
    }

    m := map[string]int{"a": 1, "b": 2}
    for k, v := range m {     // 返回 (key, value)
        fmt.Println(k, v)
    }

### Switch

分支表达式可以是任意类型，不限于常量。可省略 break，默认自动终止。

    x := []int{1, 2, 3}
    i := 2

    switch i {
    case x[1]:        // 不限于常量
        fmt.Println("a")
    case 1, 3:        // 多值匹配
        fmt.Println("b")
     default:
        fmt.Println("c")
    }

如需要继续下一分支，可使用 fallthrough，但不再判断条件。

    x := 10

    switch x {
    case 10:
        fmt.Println("a")
        fallthrough
    case 0:
        fmt.Println("b")
    }

Output:

    a
    b

省略条件表达式，可当 if...else if...else 使用。

    switch {
    case x[1] > 0:
        fmt.Println("a")
    case x[1] < 0:
        fmt.Println("b")
    default:
        fmt.Println("c")
    }

    switch i := x[2]; { // 带初始化语句
    case i > 0:
        fmt.Println("a")
    case i < 0:
        fmt.Println("b")
    default:
        fmt.Println("c")
    }

Data Structures
===============

### Array

    var a [4]int
    fmt.Println(a)              // [0 0 0 0]
    fmt.Println(len(a))          // 4
    fmt.Println(cap(a))          // 4

    var a = [2]string{"Penn", "Teller"}
    fmt.Println(a)              // [Penn Teller]
    fmt.Println(len(a))          // 2

    var a = [...]string{"Penn", "Teller"}
    fmt.Println(a)              // [Penn Teller]
    fmt.Println(len(a))          // 2

**Similar to C but with differencies:**\
Go's arrays are values. An array variable denotes the entire array; it
is not a pointer to the first array element (as would be the case in C).

### Slice

The type specification for a slice:

    []T

Slices build on arrays to provide great power and convenience. Unlike an
array type, a slice type has no specified length.\
Create with literal:

    var s = []int{0, 1, 2, 3, 4}
    fmt.Println(s)              // [0 1 2 3 4]
    fmt.Println(len(s))         // 5
    fmt.Println(cap(s))         // 5

Create with built-in function make:

    func make([]T, len, cap) []T

    var s = make([]int, 5, 10)
    fmt.Println(s)              // [0 0 0 0 0]
    fmt.Println(len(s))         // 5
    fmt.Println(cap(s))         // 10

The zero value of a slice is nil.

    var s []int
    fmt.Println(s)             // []
    fmt.Println(s == nil)      // true
    fmt.Println(len(s))        // 0

Create by "slicing" an existing slice or array.

    b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
    // b[1:4] == []byte{'o', 'l', 'a'}
    // b[:2] == []byte{'g', 'o'}
    // b[2:] == []byte{'l', 'a', 'n', 'g'}
    // b[:] == b

Create a slice given an array:

    b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
    s := b[:]  // a slice referencing the storage of b

make(\[\]byte, 5), is structured like this:\

Re-slicing a slice doesn't make a copy of the underlying array.\

    t := make([]byte, len(s), (cap(s)+1)*2)
    copy(t, s)
    s = t

Grow a slice by built-in function append:

    func append(s []T, x ...T) []T 

    a := make([]int, 1)
    // a == []int{0}
    a = append(a, 1, 2, 3)
    // a == []int{0, 1, 2, 3}

Append one slice to another:

    a := []string{"John", "Paul"}
    b := []string{"George", "Ringo", "Pete"}
    a = append(a, b...) // equivalent to "append(a, b[0], b[1], b[2])"
    // a == []string{"John", "Paul", "George", "Ringo", "Pete"}

### Map

A map is an unordered collection of key-value pairs.

    var x map[string]int

x is a map of strings to ints.

Maps have to be initialized before they can be used.\\

    x := make(map[string]int)
    x["10"] = 10
    fmt.Println(x["10"])

Delete items from a map using the built-in delete function:

    delete(x, “10”)

Looking up a key which doesn't exist returns the zero value.

    fmt.Println(x["unexisted"])  // 0

Similar to C++ std::map.\
Accessing an element of a map can return two values instead of just one.

The first value is the result of the lookup,\
The second tells us whether or not the lookup was successful.

In Go we often see code like this:

    if value, ok := x["0"]; ok {
        fmt.Println(value, ok)
    }

Shorter way to create maps:

    x := map[string]int{
        "1": 1,
        "2": 2,
        "3": 3,
    }

### Set

No build-in support.

    map[X]struct{}

Functions
=========

    func main() {
        xs := []float64{98, 93, 77, 82, 83}
        fmt.Println(average(xs))
    }

    func average(xs []float64) float64 {
        total := 0.0
        for _, v := range xs {
            total += v
        }
        return total / float64(len(xs))
    }

We can also name the return type:

    func average(xs []float64) (avg float64) {
        total := 0.0
        for _, v := range xs {
            total += v
        }
        avg = total / float64(len(xs))
        return
    }

### Returning Multiple Values

    func f() (int, int) {
        return 5, 6
    }

    func main() {
        x, y := f()
    }

Multiple values are often used to return an error value along with the
result, or a boolean to indicate success.\
Here is a simple example, opening a file and reading some of it.

    file, err := os.Open("file.go")  // For read access.
    if err != nil {
        log.Fatal(err)
    }

    count, err := file.Read(data)
    if err != nil {
        log.Fatal(err)
    }

Here's the implementation of method Read:

    // Read reads up to len(b) bytes from the File.
    // It returns the number of bytes read and an error, if any.
    // EOF is signaled by a zero count with err set to io.EOF.
    func (f *File) Read(b []byte) (n int, err error) {
        if f == nil {
            return 0, ErrInvalid
        }
        n, e := f.read(b)
        if n < 0 {
            n = 0
        }
        if n == 0 && len(b) > 0 && e == nil {
            return 0, io.EOF
        }
        if e != nil {
            err = &PathError{"read", f.name, e}
        }
        return n, err
    }

### Variadic Functions

There is a special form available for the last parameter in a Go
function:

    func add(args ...int) int {  // Zero or more
        total := 0
        for _, v := range args {
            total += v
        }
        return total
    }

    func main() {
        fmt.Println(add(1,2,3))
    }

This is precisely how the fmt.Println function is implemented:

    func Println(a ...interface{}) (n int, err error)

We can also pass a slice of ints by following the slice with ...:

    func main() {
        xs := []int{1,2,3}
        fmt.Println(add(xs...))
    }

### Closure

It is possible to create functions inside of functions:

    func main() {
        add := func(x, y int) int {
            return x + y
        }
        fmt.Println(add(1,1))
    }

It also has access to other local variables:

    func main() {
        x := 0
        increment := func() int {
            x++
            return x
        }
        fmt.Println(increment())  // 1
        fmt.Println(increment())  // 2
    }

A function like this together with the non-local variables it references
is known as a closure.

### Recursion

    func factorial(x uint) uint {
        if x == 0 {
            return 1
        }
        return x * factorial(x-1)
    }

### Defer, Panic & Recover

    func first() {
        fmt.Println("1st")
    }
    func second() {
        fmt.Println("2nd")
    }
    func main() {
        defer second()
        first()
    }

Defer is often used when resources need to be freed in some way.

    f, _ := os.Open(filename)
    defer f.Close()

### Panic & Recover

    func main() {
        defer func() {
            str := recover()
            fmt.Println(str)  // PANIC
        }()
        panic("PANIC")
    }

A panic generally indicates a programmer error (for example,

attempting to access an index of an array that's out of bounds\
forgetting to initialize a map\
etc.

) or an exceptional condition that there's no easy way to recover from.

    // pkg\bytes\buffer.go

    // makeSlice allocates a slice of size n. If the allocation fails,
    // it panics with ErrTooLarge.
    func makeSlice(n int) []byte {
        // If the make fails, give a known error.
        defer func() {
            if recover() != nil {
                panic(ErrTooLarge)
            }
        }()
        return make([]byte, n)
    }

Pointers
========

Go is similar to C in pointers.

    func zero(xPtr *int) {  // pointer
        *xPtr = 0  // dereference
    }

    func main() {
        x := 5
        zero(&x)  // address
        fmt.Println(x)  // 0
    }

### Build-in function “new”

Function “new” takes a type as an argument, allocates enough memory to
fit a value of that type and returns a pointer to it.

    func one(xPtr *int) {
        *xPtr = 1
    }

    func main() {
        xPtr := new(int)
        one(xPtr)
        fmt.Println(*xPtr)  // 1
    }

Pointers are rarely used with Go's built-in types. They are extremely
useful when paired with structs.

Structs
=======

A struct is a type which contains named fields.

    type Circle struct {
        x float64
        y float64
        r float64
    }

The fields with the same type can be collapsed:

    type Circle struct {
        x, y, r float64
    }

### Initialization

Create a local variable with zero initialization:

    var c Circle  // {0 0 0}

Use new function:

    c := new(Circle)  // Also {0 0 0}

This allocates memory for all the fields, sets each of them to their
zero value and returns a pointer (\*Circle).\
More often we want to give each of the fields a value.

    c := Circle{x: 0, y: 0, r: 5}

Or,

    c := Circle{0, 0, 5}

Note that variable c is NOT a pointer of Circle.

    func zeroCircle(c *Circle) {
        c.x = 0  // No -> operator like C! Just use dot.
        c.y = 0
        c.r = 0
    }

    func main() {
        c := Circle{x: 0, y: 0, r: 5}
        zeroCircle(&c)  // c is not a pointer, have to address it.
        fmt.Println(c)  // {0 0 0}
    }

### Constructor?

没有构造和析构方法，通常用简单工厂模式返回对象实例。

    type Queue struct {
        elements []interface{}
    }
    func NewQueue() *Queue {
        return &Queue{make([]interface{}, 10)}
    }

    // src\pkg\container\list\list.go

    package list

    // New returns an initialized list.
    func New() *List { return new(List).Init() }

### Methods

Normal function:

    func circleArea(c *Circle) float64 {
        return math.Pi * c.r*c.r
    }

Method:

    func (c *Circle) area() float64 {  // Receiver
        return math.Pi * c.r*c.r
    }

Call the method:

    fmt.Println(c.area())

### Interfaces

Similarily, define struct Rectangle:

    type Rectangle struct {
        x1, y1, x2, y2 float64
    }

    func (r *Rectangle) area() float64 {
        l := distance(r.x1, r.y1, r.x1, r.y2)
        w := distance(r.x1, r.y1, r.x2, r.y1)
        return l * w
    }

Both Circle and Rectangle have a method named area(). Let's define an
Interface for this similarity:

    type Shape interface {
        area() float64
    }

Instead of defining fields, interface defines a “method set”: a list of
methods that a type must have in order to “implement” the interface.\
Function taking interface types as arguments:

    func totalArea(shapes ...Shape) float64 {
        var area float64
        for _, s := range shapes {
            area += s.area()
        }
        return area
    }

Now call it:

    fmt.Println(totalArea(&c, &r))

Interfaces can also be used as fields:

    type MultiShape struct {
        shapes []Shape
    }

We can even turn MultiShape itself into a Shape by giving it an area
method:

    func (m *MultiShape) area() float64 {
        var area float64
        for _, s := range m.shapes {
            area += s.area()
        }
        return area
    }

Packages
========

编译工具对源码目录有严格要求，每个工作空间 (workspace) 必须由
bin、pkg、src 三个目录组成。

“D:\\proj\\Go” is NOT in \$GOPATH.

    $ D:\proj\Go>go build hello.go

    $ D:\proj\Go>go install
    go install: no install location for directory D:\proj\Go outside GOPATH

**所有代码都必须组织在 package 中：**\
源文件头部以 "package &lt;name&gt;" 声明包名称。\
包由同一目录下的多个源码文件组成。\
包名类似 namespace，与包所在目录名、编译文件名无关。\
可执行文件必须包含 package main，入口函数 main。

### Access Control

Lower case private, upper case public.

    package list

    type Element struct {
        next, prev *Element  // private
        list *List           // private
        Value interface{}    // public
    }

    package csi

    type Patient struct {
        Id string
        Name string
        BirthDate time.Time
        dose float32            // Won't exported.
    }

    package main
    ...
    p := &csi.Patient{"1", "Adam", newDate(1984, time.March, 26), 1.0}

    Error:
    > implicit assignment of unexported field 'dose' in csi.Patient literal

Concurrency
===========

Go has rich support for concurrency using goroutines and channels.

### Goroutines

A goroutine is a function that is capable of running concurrently with
other functions.

    func f(n int) {
        for i := 0; i < 10; i++ {
            fmt.Println(n, ":", i)
        }
    }

    func main() {
        go f(0)
        var input string
        fmt.Scanln(&input)
    }

Goroutines are lightweight, you can create many of them:

    func main() {
        for i := 0; i < 10; i++ {
            go f(i)
        }
        var input string
        fmt.Scanln(&input)
    }

### Channels

Channels provide a way for two goroutines to communicate with one
another and synchronize their execution.

    func pinger(c chan string) {
        for i := 0; ; i++ {
            c <- "ping"
        }
    }

    func printer(c chan string) {
        for {
            msg := <- c
            fmt.Println(msg)
            time.Sleep(time.Second * 1)
        }
    }

    func main() {
        var c chan string = make(chan string)
        go pinger(c)
        go printer(c)
        var input string
        fmt.Scanln(&input)
    }

When pinger attempts to send a message on the channel it will wait until
printer is ready to receive the message. (this is known as blocking)

Create Library
==============

Choose a package path (carestream.com/dental/csi) and create the package
dir:

      $GOPATH\src\carestream.com\dental\csi

Next, create a file named patient.go, containing the following code.

    package csi

    import "time"

    type Patient struct {
        Id string
        Name string
        BirthDate time.Time
        Dose float32
    }

Now, build it:

    $ go build carestream.com\dental\csi

No output. Do go install for it.

    $ go install carestream.com\dental\csi

Get:

    $GOPATH\pkg\windows_amd64\carestream.com\dental\csi.a

**THE END**

