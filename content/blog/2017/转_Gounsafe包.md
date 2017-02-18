
---
date: 2017-02-18T11:13:48+08:00
title: "Gounsafe包"
description: ""
disqus_identifier: 1487387628447131649
slug: "Go-unsafebao"
source: "https://my.oschina.net/xinxingegeya/blog/841058"
tags: 
- Go1.8 
topics:
- 编程语言与开发
---

Go unsafe包

unsafe包概述
============

直到现在（Go1.7），unsafe包含以下资源：

三个函数：

    // unsafe.Sizeof函数返回操作数在内存中的字节大小,参数可以是任意类型的表达式,但是它并不会对表达式进行求值.
    // 一个Sizeof函数调用是一个对应uintptr类型的常量表达式,
    // 因此返回的结果可以用作数组类型的长度大小，或者用作计算其他的常量.
    func Sizeof(x ArbitraryType) uintptr

    //函数的参数必须是一个字段 x.f, 然后返回 f 字段相对于 x 起始地址的偏移量, 包括可能的空洞.
    func Offsetof(x ArbitraryType) uintptr

    //unsafe.Alignof 函数返回对应参数的类型需要对齐的倍数.
    func Alignof(x ArbitraryType) uintptr

> 内存空洞是编译器自动添加的没有被使用的内存空间，用于保证后面每个字段或元素的地址相对于结构或数组的开始地址能够合理地对齐（译注：内存空洞可能会存在一些随机数据，可能会对用unsafe包直接操作内存的处理产生影响）。

和一种类型：

    type Pointer *ArbitraryType

这里，ArbitraryType不是一个真正的类型。官方导出这个类型只是出于完善文档的考虑，在其他的库和任何项目中都没有使用价值，除非程序员故意使用它。

 

unsafe.Sizeof, Alignof 和 Offsetof
==================================

> 计算机在加载和保存数据时，如果内存地址合理地对齐的将会更有效率。例如2字节大小的int16类型的变量地址应该是偶数，一个4字节大小的rune类型变量的地址应该是4的倍数，一个8字节大小的float64、uint64或64-bit指针类型变量的地址应该是8字节对齐的。但是对于再大的地址对齐倍数则是不需要的，即使是complex128等较大的数据类型最多也只是8字节对齐。
>
>  
>
> ***由于地址对齐这个因素，一个聚合类型（结构体或数组）的大小至少是所有字段或元素大小的总和，或者更大因为可能存在内存空洞。内存空洞是编译器自动添加的没有被使用的内存空间，用于保证后面每个字段或元素的地址相对于结构或数组的开始地址能够合理地对齐***（译注：内存空洞可能会存在一些随机数据，可能会对用unsafe包直接操作内存的处理产生影响）。

一个结构体变量 x 以及其在64位机器上的典型的内存. 灰色区域是空洞.

    var x struct {
        a bool
        b int16
        c []int
    }

![](https://static.oschina.net/uploads/space/2017/0217/165238_zUMW_1469576.png)

对结构体变量的三个字段调用unsafe包相关函数的计算结果如下，

    package main

    import (
        "fmt"
        "unsafe"
    )

    func main() {

        var x struct {
            a bool
            b int16
            c []int
        }

        //通常情况下布尔和数字类型需要对齐到它们本身的大小(最多8个字节),其它的类型对齐到机器字大小.(64位的机器字大小为64位,8字节)

        fmt.Printf("%-30s%-30s%-30s%-50s\n",
            "Row", "Sizeof", "Alignof(对齐倍数)", "Offsetof(偏移量)")

        fmt.Printf("%-30s%-30d%-30d%-50s\n",
            "x", unsafe.Sizeof(x), unsafe.Alignof(x), "")
        fmt.Printf("%-30s%-30d%-30d%-50d\n",
            "x.a", unsafe.Sizeof(x.a), unsafe.Alignof(x.a), unsafe.Offsetof(x.a))
        fmt.Printf("%-30s%-30d%-30d%-50d\n",
            "x.b", unsafe.Sizeof(x.b), unsafe.Alignof(x.b), unsafe.Offsetof(x.b))
        fmt.Printf("%-30s%-30d%-30d%-50d\n",
            "x.c", unsafe.Sizeof(x.c), unsafe.Alignof(x.c), unsafe.Offsetof(x.c))
    }

运行结果，

    Row                           Sizeof                        Alignof(对齐倍数)                 Offsetof(偏移量)                                     
    x                             32                            8                                                                               
    x.a                           1                             1                             0                                                 
    x.b                           2                             2                             2                                                 
    x.c                           24                            8                             8                            

 

unsafe.Pointer
==============

大多数指针类型会写成\*T，表示是“一个指向T类型变量的指针”。unsafe.Pointer是特别定义的一种指针类型（译注：类似C语言中的void\*类型的指针），它可以包含任意类型变量的地址。当然，我们不可以直接通过\*p来获取unsafe.Pointer指针指向的真实变量的值，因为我们并不知道变量的具体类型。和普通指针一样，unsafe.Pointer指针也是可以比较的，并且支持和nil常量比较判断是否为空指针。

一个普通的\*T类型指针可以被转化为unsafe.Pointer类型指针，并且一个unsafe.Pointer类型指针也可以被转回普通的指针，被转回普通的指针类型并不需要和原始的\*T类型相同。通过将\*float64类型指针转化为\*uint64类型指针，我们可以查看一个浮点数变量的位模式。

    package main

    import (
        "fmt"
        "unsafe"
        "reflect"
    )

    func Float64bits(f float64) uint64 {
        fmt.Println(reflect.TypeOf(unsafe.Pointer(&f)))  //unsafe.Pointer
        fmt.Println(reflect.TypeOf((*uint64)(unsafe.Pointer(&f))))  //*uint64
        return *(*uint64)(unsafe.Pointer(&f))
    }

    func main() {
        fmt.Printf("%#016x\n", Float64bits(1.0)) // "0x3ff0000000000000"
    }

通过转为新类型指针，我们可以更新浮点数的位模式。通过位模式操作浮点数是可以的，***但是更重要的意义是指针转换语法让我们可以在不破坏类型系统的前提下向内存写入任意的值。***

一个unsafe.Pointer指针也可以被转化为uintptr类型，然后保存到指针型数值变量中（译注：这只是和当前指针相同的一个数字值，并不是一个指针），然后用以做必要的指针数值运算。（第三章内容，uintptr是一个无符号的整型数，足以保存一个地址）这种转换虽然也是可逆的，但是将uintptr转为unsafe.Pointer指针可能会破坏类型系统，因为并不是所有的数字都是有效的内存地址。

许多将unsafe.Pointer指针转为原生数字，然后再转回为unsafe.Pointer类型指针的操作也是不安全的。比如下面的例子需要将变量x的地址加上b字段地址偏移量转化为\*int16类型指针，然后通过该指针更新x.b：

    package main

    import (
        "fmt"
        "unsafe"
    )

    func main() {

        var x struct {
            a bool
            b int16
            c []int
        }

        // 和 pb := &x.b 等价
        pb := (*int16)(unsafe.Pointer(
            uintptr(unsafe.Pointer(&x)) + unsafe.Offsetof(x.b)))
        *pb = 42
        fmt.Println(x.b) // "42"
    }

上面的写法尽管很繁琐，但在这里并不是一件坏事，因为这些功能应该很谨慎地使用。不要试图引入一个uintptr类型的临时变量，因为它可能会破坏代码的安全性（译注：这是真正可以体会unsafe包为何不安全的例子）。下面段代码是错误的：

    package main

    import (
        "fmt"
        "unsafe"
    )

    func main() {

        var x struct {
            a bool
            b int16
            c []int
        }

        // NOTE: subtly incorrect!
        tmp := uintptr(unsafe.Pointer(&x)) + unsafe.Offsetof(x.b)
        pb := (*int16)(unsafe.Pointer(tmp))
        *pb = 42
        fmt.Println(x.b) // "42"
    }

产生错误的原因很微妙。有时候垃圾回收器会移动一些变量以降低内存碎片等问题。这类垃圾回收器被称为移动GC。当一个变量被移动，所有的保存改变量旧地址的指针必须同时被更新为变量移动后的新地址。从垃圾收集器的视角来看，一个unsafe.Pointer是一个指向变量的指针，因此当变量被移动是对应的指针也必须被更新；但是uintptr类型的临时变量只是一个普通的数字，所以其值不应该被改变。上面错误的代码因为引入一个非指针的临时变量tmp，导致垃圾收集器无法正确识别这个是一个指向变量x的指针。当第二个语句执行时，变量x可能已经被转移，这时候临时变量tmp也就不再是现在的&x.b地址。第三个向之前无效地址空间的赋值语句将彻底摧毁整个程序！

===================

unsafe.Pointer的使用规则，

> （1）任何类型的指针都可以被转化为Pointer
>
> （2）Pointer可以被转化为任何类型的指针
>
> （3）uintptr可以被转化为Pointer
>
> （4）Pointer可以被转化为uintptr

举个例子：

    package main

    import (
        "unsafe"
        "fmt"
    )

    func main() {
        var n int64 = 5
        var pn = &n
        var pf = (*float64)(unsafe.Pointer(pn))
        fmt.Println(*pf) //2.5e-323
        *pf = 3.1415
        fmt.Println(n) //4614256447914709615
    }

在这个例子中的转换可能是无意义的，但它是安全和合法的。

uintptr 和 unsafe.Pointer 的互相转换，

    package main

    import (
        "unsafe"
        "fmt"
    )

    func main() {
        a := [4]int{0, 1, 2, 3}
        p := &a[1] // 内存地址
        p1 := unsafe.Pointer(p) 
        p2 := uintptr(p1)
        p3 := unsafe.Pointer(p2)
        fmt.Println(p1) // 0xc420014208
        fmt.Println(p2) // 842350543368
        fmt.Println(p3) // 0xc420014208
    }

 

uintptr
=======

关于 uintptr ,

    // uintptr is an integer type that is large enough to hold the bit pattern of
    // any pointer.
    type uintptr uintptr

uintptr 的底层实现如下，在\$GOROOT/src/pkg/runtime/runtime.h中，

    #ifdef _64BIT
    typedef uint64          uintptr;
    typedef int64           intptr;
    typedef int64           intgo; // Go's int
    typedef uint64          uintgo; // Go's uint
    #else
    typedef uint32          uintptr;
    typedef int32           intptr;
    typedef int32           intgo; // Go's int
    typedef uint32          uintgo; // Go's uint
    #endif

uintptr和intptr是无符号和有符号的指针类型，并且确保在64位平台上是8个字节，在32位平台上是4个字节，uintptr主要用于golang中的指针运算。

 

合法用例1：在[]T和[]MyT之间转换
===============================

在这个例子里，我们用int作为T：

    type MyInt int

在Golang中，[]int 和 []MyInt是两种不同的类型。
因此，[]int的值不能转换为[]MyInt，反之亦然。
但是在unsafe.Pointer的帮助下，转换是可能的：

    package main

    import (
        "unsafe"
        "fmt"
    )

    func main() {
        type MyInt int

        a := []MyInt{0, 1, 2}
        // b := ([]int)(a) // error: cannot convert a (type []MyInt) to type []int
        b := *(*[]int)(unsafe.Pointer(&a))

        b[0] = 3

        fmt.Println("a =", a) // a = [3 1 2]
        fmt.Println("b =", b) // b = [3 1 2]

        a[2] = 9

        fmt.Println("a =", a) // a = [3 1 9]
        fmt.Println("b =", b) // b = [3 1 9]
    }

 

合法用例2: 调用sync/atomic包中指针相关的函数
============================================

sync /
atomic包中的以下函数的大多数参数和结果类型都是unsafe.Pointer或\*unsafe.Pointer：

    // CompareAndSwapPointer executes the compare-and-swap operation for a unsafe.Pointer value.
    func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool)

    // LoadPointer atomically loads *addr.
    func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)

    // StorePointer atomically stores val into *addr.
    func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)

    // SwapPointer atomically stores new into *addr and returns the previous *addr value.
    func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer)

要使用这些功能，必须导入unsafe包。

    package main

    import (
        "unsafe"
        "fmt"
        "sync/atomic"
        "time"
        "sync"
        "log"
        "math/rand"
    )

    var data *string

    func Data() string {
        p := (*string)(atomic.LoadPointer((*unsafe.Pointer)(unsafe.Pointer(&data))))
        if p == nil {
            return ""
        } else {
            return *p
        }
    }

    // set data atomically
    func SetData(d string) {
        atomic.StorePointer((*unsafe.Pointer)(unsafe.Pointer(&data)), unsafe.Pointer(&d))
    }

    func main() {

        var wg sync.WaitGroup
        wg.Add(200)

        for range [100]struct{}{} {
            go func() {
                time.Sleep(time.Second * time.Duration(rand.Intn(1000)) / 1000)

                log.Println(Data())
                wg.Done()
            }()
        }

        for i := range [100]struct{}{} {
            go func(i int) {
                time.Sleep(time.Second * time.Duration(rand.Intn(1000)) / 1000)
                s := fmt.Sprint("#", i)
                log.Println("====", s)

                SetData(s)
                wg.Done()
            }(i)
        }

        wg.Wait()

        fmt.Println("final data = ", *data)
    }

转载：

https://shifei.me/gopl-zh/ch13/ch13-02.html

http://www.open-open.com/lib/view/open1391347613192.html

=======END=======

