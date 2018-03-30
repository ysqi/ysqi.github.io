
---
date: 2016-12-31T11:34:34+08:00
title: "Go语法"
description: ""
disqus_identifier: 1485833674195445223
slug: "Goyu-fa"
source: "https://segmentfault.com/a/1190000002623278"
tags: 
- golang 
topics:
- 编程语言与开发
---

Go基础
======

变量
----

基本结构：`var 变量名 变量类型 = 值`\
注：`_`（下划线）是个特殊的变量名，任何赋予它的值都会被丢弃

    package main

    /* 全局变量 */
    // 仅声明, 必要有var和变量类型
    var a int
    var b, c int

    // 声明并初始化，变量类型可省略
    var d int = 1
    var e, f int = 1, 2
    var g = 1 // 自动推断类型
    var h, i = 1, "string" // 类型可以不一样

    func main() {
        /* 局部变量特有的声明方式 */
        j := 1;
        k, l := 1, 2
    }

常量
----

常量可定义为数值、布尔值或字符串等类型。

    /* 全局和局部声明方式相同 */
    const a int = 1
    const b = 1
    const c, d = 1, 2 "string" // 类型可以不一样

内置基本类型
------------

### Boolean

布尔值的类型为bool，值是true或false，默认为false。

注：不能用0和非0表示true或false

### 数值类型

    1. 整型 
        * 分为无符号和带符号，例如：int和uint
        * 8，16，32，64位，例如：int32和uint32
        * rune是int32的别称，byte是uint8的别称
    2. 浮点型 float32和float64
    3. 复数 complex64和complex128
    注：不同类型之间不能进行运算

### 字符串

#### 定义

    var a string 
    var b string = "" 
    func test() {
        no, yes, maybe := "no", "yes" 
    }

#### 修改

    s := "hello"
    c := []byte(s)  // 将字符串 s 转换为 []byte 类型
    c[0] = 'c'
    s2 := string(c)  // 再转换回 string 类型
    fmt.Printf("%s\n", s2)

#### 连接

    s := "hello,"
    m := " world"
    a := s + m
    fmt.Printf("%s\n", a)

#### 原始格式输出

\`
括起的字符串为Raw字符串，即字符串在代码中的形式就是打印时的形式，它没有字符转义，换行也将原样输出。

    m := `hello
        world`

错误类型
--------

Go内置有一个error类型，专门用来处理错误信息，Go的package里面还专门有一个包errors来处理错误：

    err := errors.New("emit macho dwarf: elf header corrupted")
    if err != nil {
        fmt.Print(err)
    }

Go数据底层的存储
----------------

下面这张图来源于Russ Cox
Blog中一篇介绍Go数据结构的文章，大家可以看到这些基础类型底层都是分配了一块内存，然后存储了相应的值。

一些技巧
--------

### 分组声明

    import(
        "fmt"
        "os"
    )

    const(
        i = 100
        pi = 3.1415
        prefix = "Go_"
    )

    var(
        i int
        pi float32
        prefix string
    )

### iota枚举

Go里面有一个关键字iota，这个关键字用来声明enum的时候采用，它默认开始值是0，每调用一次加1：

    const(
        x = iota  // x == 0
        y = iota  // y == 1
        z = iota  // z == 2
        w  // 常量声明省略值时，默认和之前一个值的字面相同。这里隐式地说w = iota，因此w == 3。其实上面y和z可同样不用"= iota"
    )

    const v = iota // 每遇到一个const关键字，iota就会重置，此时v == 0

    const ( 
      e, f, g = iota, iota, iota //e=0,f=0,g=0 iota在同一行值相同
    )

> 除非被显式设置为其它值或iota，每个const分组的第一个常量被默认设置为它的0值，第二及后续的常量被默认设置为它前面那个常量的值，如果前面那个常量的值是iota，则它也被设置为iota。

### 私有和公有

大写字母开头的变量或函数，为公有，相当于java中的public\
小写字母开头的变量或函数为，私有，相当于java中的private。

array、slice、map
-----------------

### array

基本结构：`var 变量名 [长度]类型`

-   数组长度不能改变
-   长度也是数组类型的一部分，因此[3](http://www.tuicool.com/articles/QrymYz)int与\[4\]int是不同的类型
-   当把一个数组作为参数传入函数的时候，传入的其实是该数组的副本，而不是它的指针。

<!-- -->

    var arr [10]int  // 声明了一个int类型的数组
    a := [3]int{1, 2, 3} // 声明了一个长度为3的int数组
    b := [10]int{1, 2, 3} // 声明了一个长度为10的int数组，其中前三个元素初始化为1、2、3，其它默认为0
    c := [...]int{4, 5, 6} // 可以省略长度而采用`...`的方式，Go会自动根据元素个数来计算长度

### slice

#### 声明

基本结构：`var 变量名 []类型`

slice是一个引用类型。slice总是指向一个底层array，slice的声明也可以像array一样，只是不需要长度。

    <!-- 直接声明 -->
    var slice []int
    var slice = []int{1, 2, 3}
    slice := []byte {'a', 'b', 'c'}
    <!-- 从数组中截取 -->
    array := [3]byte {'a', 'b', 'c'}
    slice := array[1, 2] // slice通过array[i:j]来获取，其中i是数组的开始位置，j是结束位置，但不包含array[j]，它的长度是j-i。 

#### 内置函数

-   len 获取slice的长度
-   cap 获取slice的最大容量
-   append
    向slice里面追加一个或者多个元素，然后返回一个和slice一样类型的slice
-   copy
    函数copy从源slice的src中复制元素到目标dst，并且返回复制的元素的个数

#### 长度与容量

    a := [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 0}
    s := a[0:]
    s = append(s, 11, 22, 33)
    sa := a[2:7]
    sb := sa[3:5]
    fmt.Println(a, len(a), cap(a))    //输出：[1 2 3 4 5 6 7 8 9 0] 10 10
    fmt.Println(s, len(s), cap(s))    //输出：[1 2 3 4 5 6 7 8 9 0 11 22 33] 13 20
    fmt.Println(sa, len(sa), cap(sa)) //输出：[3 4 5 6 7] 5 8
    fmt.Println(sb, len(sb), cap(sb)) //输出：[6 7] 2 5

-   长度为已存放个数，容量为可存放个数
-   对数组来说，长度和容量总是相等的
-   slice的容量可以大于长度，如果容量不足，将动态分配新的数组空间
-   array\[i:j:k\]，k - i为容量，k默认为数组长度

#### 陷阱

当Slice的容量还有空闲的时候，append进来的元素会直接使用空闲的容量空间，但是一旦append进来的元素个数超过了原来指定容量值的时候，内存管理器就是重新开辟一个更大的内存空间，用于存储多出来的元素，并且会将原来的元素复制一份，放到这块新开辟的内存空间。

    a := []int{1, 2, 3, 4}
    sa := a[1:3]
    fmt.Printf("%p\n", sa) //输出：0xc0840046e0
    sa = append(sa, 11, 22, 33)
    fmt.Printf("%p\n", sa) //输出：0xc084003200

[参考链接](http://www.tuicool.com/articles/QrymYz)

### map

#### 声明

基本结构：`map[keyType]valueType`

    m1 := make(map[string]string)
    m1["aa"] = "bb" // 添加
    m2 := map[string]float32{"C":5, "Go":4.5, "Python":4.5, "C++":2 }
    m2["C++"] = 5 // 修改
    delete(m2, "C")  // 删除key为C的元素
    len(m2) // 长度

#### 特点

-   map是无序的，每次打印出来的map都会不一样，它不能通过index获取，而必须通过key获取
-   map的长度是不固定的，也就是和slice一样，也是一种引用类型
-   map和其他基本型别不同，它不是thread-safe，在多个go-routine存取时，必须使用mutex
    lock机制
-   map\[key\]，有两个返回值，第一个是value，第二个是是否存在对应的值

### make、new操作

make用于内建类型（map、slice
和channel）的内存分配。new用于各种类型的内存分配。

-   new返回的是指针
-   make返回初始化后的（非零）值

### 零值

关于“零值”，所指并非是空值，而是一种“变量未填充前”的默认值，通常为0。
此处罗列 部分类型 的 “零值”

    int     0
    int8    0
    int32   0
    int64   0
    uint    0x0
    rune    0 //rune的实际类型是 int32
    byte    0x0 // byte的实际类型是 uint8
    float32 0 //长度为 4 byte
    float64 0 //长度为 8 byte
    bool    false
    string  ""

流程与函数
----------

### 流程控制

Go中流程控制分三大类：条件判断，循环控制和无条件跳转

#### if

Go的if有一个强大的地方就是条件判断语句里面允许声明一个变量，这个变量的作用域只能在该条件逻辑块内，其他地方就不起作用了，如下所示

    // 计算获取值x,然后根据x返回的大小，判断是否大于10。
    if x := computedValue(); x > 10 {
        fmt.Println("x is greater than 10")
    } else {
        fmt.Println("x is less than 10")
    }

    //这个地方如果这样调用就编译出错了，因为x是条件里面的变量
    fmt.Println(x)

#### goto

Go有goto语句——请明智地使用它。用goto跳转到必须在当前函数内定义的标签(大小写敏感)。例如假设这样一个循环：

    func myFunc() {
        i := 0
    Here:   //这行的第一个词，以冒号结束作为标签
        println(i)
        i++
        goto Here   //跳转到Here去
    }

#### for

Go里面最强大的一个控制逻辑就是for，它即可以用来循环读取数据，又可以当作while来控制逻辑，还能迭代操作。它的语法如下：

    /* expression1和expression3是变量声明或者函数调用返回值之类的，expression2是用来条件判断，expression1在循环开始之前调用，expression3在每轮循环结束之时调用。 */
    for expression1; expression2; expression3 {
        //...
    }

    /* 例如 */
    for index:= 0; index < 10 ; index++ {
    }
    // 平行赋值
    for a, b:= 0, 0; b < 10 ; b++ {
    }

##### while语句

    /* while语句 */
    for ; sum < 1000;  {
        sum += sum
    }
    // ;可省略
    for sum < 1000 {
        sum += sum
    }

##### break和continue

    /* break和continue与java一致 */
    for index := 10; index>0; index-- {
        if index == 5{
            break // 或者continue
        }
        fmt.Println(index)
    }
    // break打印出来10、9、8、7、6
    // continue打印出来10、9、8、7、6、4、3、2、1

##### range

    /* for配合range可以用于读取slice和map的数据 */
    // 第一个返回值是key，第二个返回值是value
    // 如果是slice，那么key为下标
    for k,v:=range map {
        fmt.Println("map's key:",k)
        fmt.Println("map's val:",v)
    }

#### switch

与java不同的是，默认每个`case`执行完毕后会跳出`switch语句`，如果希望继续执行下一个`case`，需要加入`fallthrough`关键字

    i := 10
    switch i {
    case 1:
        fmt.Println("i is equal to 1")
    case 2, 3, 4:
        fmt.Println("i is equal to 2, 3 or 4")
    case 10:
        fmt.Println("i is equal to 10")
        fallthrough
    default:
        fmt.Println("------------------------")
    }

### 函数

    /* 基本结构 */
    func funcName(input1 type1, input2 type2) (output1 type1, output2 type2) {
        //这里是处理逻辑代码
        //返回多个值
        return output1, output2
    }

    /* output1和output2，可以不声明，但必须注明返回类型*/
    func funcName(input1 type1, input2 type2) (type1, type2) {

        return "", "" 
    }

    /* 如果只有一个返回值且不声明返回值变量，可以这样写 */
    func funcName(input1 type1, input2 type2) type {

    }

    /* 没有返回值，可全部省略 */
    func funcName(input1 type1, input2 type2) {

    }

    /* 前后参数类型相同，可省略前面的参数类型*/
    func funcName(input1, input2 int, input3 string, input4, input5 float32) (output1, output2 int){

    }

    /* 官方建议：最好命名返回值，因为不命名返回值，虽然使得代码更加简洁了，但是会造成生成的文档可读性差。 */
    func SumAndProduct(A, B int) (add int, Multiplied int) {
        add = A+B
        Multiplied = A*B
        // 这里也是个特别的地方
        return
    }

### 变参

基本结构：`func myfunc(arg ...int) {}`

-   参数的类型全部是int
-   变量arg是一个int的slice

<!-- -->

    for _, n := range arg {
        fmt.Printf("And the number is: %d\n", n)
    }

### 传值与传指针

-   函数的参数，传入的都是copy
-   即使传入的是指针，也是指针的copy

指针的优点

-   传指针使得多个函数能操作同一个对象。
-   传指针比较轻量级
    (8bytes),只是传内存地址，我们可以用指针传递体积大的结构体。如果用参数值传递的话,
    在每次copy上面就会花费相对较多的系统开销（内存和时间）。所以当你要传递大的结构体的时候，用指针是一个明智的选择。
-   Go语言中string，slice，map这三种类型的实现机制类似指针，所以可以直接传递，而不用取地址后传递指针。（注：若函数需改变slice的长度，则仍需要取地址传递指针）

<!-- -->

    package main
    import "fmt"

    //简单的一个函数，实现了参数+1的操作
    func add1(a *int) int { // 请注意，
        *a = *a+1 // 修改了a的值
        return *a // 返回新值
    }

    func main() {
        x := 3

        fmt.Println("x = ", x)  // 应该输出 "x = 3"

        x1 := add1(&x)  // 调用 add1(&x) 传x的地址

        fmt.Println("x+1 = ", x1) // 应该输出 "x+1 = 4"
        fmt.Println("x = ", x)    // 应该输出 "x = 4"
    }

### defer

-   defer语句会在函数返回前执行

<!-- -->

    func ReadWrite() bool {
        file.Open("file")
        defer file.Close()
        if failureX {
            return false
        }
        if failureY {
            return false
        }
        return true
    }

-   defer是采用后进先出模式，所以如下代码会输出4 3 2 1 0

<!-- -->

    for i := 0; i < 5; i++ {
        defer fmt.Printf("%d ", i)
    }

### 函数作为值、类型

基本结构：`type typeName func(input1 inputType1 , input2 inputType2) (result1 resultType1)`

-   拥有相同参数列表和返回值列表的函数，属于同一函数类型。比如下面例子中的isOdd和isEven函数。

<!-- -->

    package main
    import "fmt"

    type testInt func(int) bool // 声明了一个函数类型

    func isOdd(integer int) bool {
        if integer%2 == 0 {
            return false
        }
        return true
    }

    func isEven(integer int) bool {
        if integer%2 == 0 {
            return true
        }
        return false
    }

    // 声明的函数类型在这个地方当做了一个参数

    func filter(slice []int, f testInt) []int {
        var result []int
        for _, value := range slice {
            if f(value) {
                result = append(result, value)
            }
        }
        return result
    }

    func main(){
        slice := []int {1, 2, 3, 4, 5, 7}
        fmt.Println("slice = ", slice)
        odd := filter(slice, isOdd)    // 函数当做值来传递了
        fmt.Println("Odd elements of slice are: ", odd)
        even := filter(slice, isEven)  // 函数当做值来传递了
        fmt.Println("Even elements of slice are: ", even)
    }

