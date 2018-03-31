
---
date: 2017-02-24T08:31:56+08:00
title: "Golang学习摘录一:初识"
description: ""
disqus_identifier: 1487896316288320107
slug: "Golangxue-xi-zhai-lu-yi-:chu-shi"
source: "http://www.jianshu.com/p/b655bd357019"
tags: 
- golang 
categories:
- 编程语言与开发
---

1、Hello Word编写方式
---------------------

    package main
    import "fmt"  

    func main (){
      fmt.Printf("Hello, world")
    }

编译 `go build helloworld.go`\
运行 `./helloworld`

2、声明方式
-----------

    1、普通方式
    var a int = 15
    var b bool = false
    或
    var a int
    var b bool
    a = 15
    b = false
    2、   :=会自动匹配类型，只能在函数内使用
    a := 15
    b := false
    3、中括号的形式：
    var (
          x int
          b bool
    )
    4、平行赋值
    a,b := 20,16

    #注意 :Go 的编译器对声明却未使用的变量在报错。

    5、常量，只能是数字、字符串或布尔值
    const(     // 枚举的生成方式
          a = iota   // a为0
          b = iota  // b为1，改行的 “=iota”可省略
    )
    如果需要,可以明确指定常量的类型:
    const (
            a = 0 
            b string = "0"
    )

3、字符串
---------

    var s string = "hello" 
    #Go中字符串是不可变的

如果想修改字符需要使用下面的方法

    s := "hello"
    c := []rune(s)
    c[0] = 'c'
    s2 := string(c)
    fmt.Printf("%s\n", s2)

多行字符串

    基于分号的置入,你需要小心使用多行字符串。
    s := "Starting part" +
         "Ending part"
    Go 就不会在错误的地方插入分号。另一种方式是使用反引号 ` 作为原始字符串符 号:
    s := `Starting part
         Ending part`
    留意最后一个例子 s 现在也包含换行。不像转义字符串标识 ,原始字符串标识的值 在引号内的字符是不转义的。

4、array
--------

array由[n]\<type\>定义，n标示array的长度，而\<type\>标示希望存储的类型，array为值类型。

    var arr [10]int
    arr[0] = 42
    arr[1] = 13

    a := [3]int{1,2,3}
    或
    a := [...]int{1,2,3}// Go会自动统计元素的个数
    // 多维数组
    a := [3][2]int { [2]int {1,2}, [2]int {3,4}, [2]int {5,6} }
    或
    a := [3][2]int { [...]int {1,2}, [...]int {3,4}, [...]int {5,6} }
    或简写为
    a := [3][2]int { {1,2}, {3,4}, {5,6} }

5、slice
--------

slice为引用类型

    sl := make([]int, 10)
    sl2 := []int{1,2,3,4}

创建了一个保存有 10 个元素的 slice。需要注意的是底层的 array
并无不同。slice 总 是与一个固定长度的 array 成对出现。其影响 slice
的容量和长度。

    // array[n:m] 从 array 创建了一个 slice,具有元素 n 到 m-1 
    a := [...]int {1, 2, 3, 4, 5} // 定义一个5个元素的array，序号从0到4
    s1 := a[2:4] // 从序号2至3创建slice，它包含元素3，4
    s2 := a[1:5] 
    s3 := a[:]  // 用array中所有元素创建slice，这是a[0:len(a)]的简化写法
    s4 := a[:4]
    s5 := s2[:]// 从slice s2创建slice，注意s5仍然指向array a

函数 append 向 slice s 追加零值或其他 x 值,并且返回追加后的新的、与 s
有相同类型的 slice。如果 s 没有足够的容量存储追加的值,append 分配一
个足够大的、新的 slice 来存放原有 slice 的元素和追加的值。因此,返回 的
slice 可能指向不同的底层 array。

    s0:= []int {0, 0}// 创建一个slice
    s1:= append(s0, 2)// 追加一个元素，s1==[]int{0,0,2}
    s2:= append(s1, 3, 5, 7)// 追加多个元素，s2==[]int{0,0,2,3,5,7}
     s3 := append(s2, s0...)// 追加一个slice，s3==[]int{0,0,3,4,5,7,0,0}。注意这三个点！

函数 copy 从源 slice src 复制元素到目标 dst,并且返回复制的元素的个
数。源和目标可能重 。元素复制的数量是 len(src) 和 len(dst) 中的最 小值。

    var a=[...]int{0,1,2,3,4,5,6,7}
    var s = make([]int, 6)
    n1 := copy(s, a[0:]) // n1 == 6, s == []int{0, 1, 2, 3, 4, 5} 
    n2 := copy(s, s[2:]) // n2 == 4, s == []int{2, 3, 4, 5, 4, 5}

6、map
------

map 可以认为是一个用字符串做索引的数 组(在其最简单的形式下)。一般定义
map 的方法是:map[\<from type\>]\<to type\>

    monthdays := map[string]int {
      "Jan": 31, "Feb": 28, "Mar": 31,
      "Apr": 30, "May": 31, "Jun": 30,
      "Jul": 31, "Aug": 31, "Sep": 30,
      "Oct": 31, "Nov": 30, "Dec": 31,   // 最后一个逗号是必须的
     }

留意，但只需要声明一个map的时候，使用make的形式：monthdays :=
make(map[string]int)\
当对 array、slice、string 或者 map 循环遍历的时候,range
会帮助你,每次调用,它 都会返回一个键和对应的值。

    year := 0
    for _, days := range monthdays { //键没有使用,因此用 _, days
    year += days
    }
    fmt.Printf("Numbers of days in a year: %d\n", year)

向 map 增加元素,可以这样做:\
monthdays["Undecim"] = 30 ← 添加一个月 monthdays["Feb"] = 29 ←
年时重写这个元素\
检查元素是否存在,可以使用下面的方式:

    var value int
    var present bool
    value, present = monthdays["Jan"] //如果存在,present 则有值 true
    或者更接近 Go 的方式
    v, ok := monthdays["Jan"] //“逗号 ok”形式

从 map 中移除元素:

    delete(monthdays, "Mar") // 删除”Mar” 
    //通常来说语句 delete(m, x) 会删除 map 中由 m[x] 建立的实例

