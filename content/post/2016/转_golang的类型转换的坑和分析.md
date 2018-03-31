
---
date: 2016-12-31T11:34:41+08:00
title: "golang的类型转换的坑和分析"
description: ""
disqus_identifier: 1485833681368335192
slug: "golangde-lei-xing-zhuai-huan-de-keng-he-fen-xi"
source: "https://segmentfault.com/a/1190000000705218"
tags: 
- 类型转换 
- golang 
categories:
- 编程语言与开发
---

首先，我们来看一个例子

    type Stringer interface {
        String() string
    }
    type String struct {
        data string
    }
    func (s *String) String() string {
        return s.data
    }

上面是类型，然后

    func GetString() *String {
        return nil
    }
    func CheckString(s Stringer) bool {
        return s == nil
    }
    func main() {
        println(CheckString(GetString()))
    }

你们猜答案是什么？\
当然，这么诡异的提问方式一看答案就是不合常理的false。\
在`CheckString`里面，s是不等于nil的。\
如果你觉得不可思议，那么可以继续看下去了。

需要强调的是，本篇文章仅仅适用于golang。

类型转换
--------

[官方文档](http://golang.org/ref/spec#Conversions)明确说明了怎么判断类型T是否可以转换成V，正常来说，T要转换成V必需显式声明。

    func check64(v int64){}
    func check(v int){}
    a := 5
    b := int64(10)
    check64(int64(a))
    check(int(b))
    check64(a) // panic

不过，如果在满足[Assignability](http://golang.org/ref/spec#Assignability)的情况下，就可以在没有显式声明的情况下自动进行类型转换\
其中，比较容易被忽略的是这两条：

1.  x is the predeclared identifier nil and T is a pointer, function,
    slice, map, channel, or interface type.
2.  x is an untyped constant representable by a value of type T.

<!-- -->

    const a = 5
    func check64(v int64){}
    func checkSlice(v []int){}
    check64(a) // -> check64(int64(a))
    check64(1024) // -> check64(int64(1024))
    checkSlice(nil) // -> checkSlice([]int(nil))

上面的代码是能够正确执行了,
需要注意的是，1024也是属于常量（[这里](http://golang.org/ref/spec#Integer_literals)说明了`Integer literals`是表示了一个`integer constant`）.

分析
----

当了解之后，再看我刚开始给的例子

    func GetString() *String {
        return nil
    }
    println(interface{}(GetString()))

这里扔出去的实际上是`*String(nil)`,
如果print出来，应该是一个(0x\*\*\*\*, 0x0)的值

> 需要ps的是，print打印一个interface时，第一个值为一个变量的动态类型的指针，第二个值为实际值的指针。这里为了得到动态类型，故意转换成了interface{}

然后，到`CheckString`，

    func CheckString(s Stringer) bool {
        return s == nil
    }

这里的`nil`会被转换成`Stringer(nil)`,
用print大法打印出来，结果是`(0x0, 0x0)`

是不是感到很奇怪，print打印出来的第一个是类型的指针，为什么是0？首先，一个变量会有一个动态类型和静态类型，如果不是interface的话，静态类型和动态类型是一样的，当interface时，静态类型是interface的值，而动态类型确是他储存的值的实际类型。而print打印的是动态类型，由于这里他的值为0，动态类型也肯定为0.

当这时候执行`CheckString(GetString())`的时候，这时候s便是`*String(nil)`,
而nil确是`<nil>(nil)`(空类型的空指针)，进行equals判断的时候，根据[spec](http://golang.org/ref/spec#Comparison_operators),
interface会在`dynamic type`和`dynamic value`完全一样的时候才返回true。所以这里很明显是false。

如何不踩坑
----------

虽然例子看起来很无厘头，但实际上这个比较容易踩坑的，特别是在想要作死的时候...

    type SpecError struct {
        err Error
    }
    func NewSpecError(err error) *SpecError {
        if err == nil {
            return nil
        }
        return &SpecError{err}
    }
    func (se *SpecError) Error() string {
        return se.err.Error()
    }
    func GetObjByID(id string) (*Obj, error) {
        obj, err := xxxxx(id)
        return obj, NewSpecError(err)

    }
    func main(){
        obj, err := GetObjByID("123")
        if err != nil {
            panic(err)
        }
        。。。
    }

好吧我承认只要不作死乖乖在`GetObjByID`把`if err != nil`不上就不会有这种事了...

思考
----

    type RouterString func() string
    type RouterBytes func() []byte
    func AddRouter(router interface{}) {
        switch router := router.(type) {
        case RouterString:
            println("string")
        case RouterBytes:
            println("bytes")
        default:
            println("unknown types")
        }
    }
    func main() {
        AddRouter(func() string{
            println("hello?")
        })
    }

这段代码哪里错了？怎么改？

