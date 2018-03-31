
---
date: 2016-12-31T11:33:06+08:00
title: "golang数组与切片"
description: ""
disqus_identifier: 1485833586636421174
slug: "golang-shu-zu-yu-qie-pian"
source: "https://segmentfault.com/a/1190000006824533"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

### 通过下面几个问题来更好理解golang 的数组和切片

1.  类型

    数组是值类型，将一个数组赋值给另一个数组时，传递的是一份拷贝。\
    切片是引用类型，切片包装的数组称为该切片的底层数组。\
    我们来看一段代码

        //a是一个数组，注意数组是一个固定长度的，初始化时候必须要指定长度，不指定长度的话就是切片了
        a := [3]int{1, 2, 3}
        //b是数组，是a的一份拷贝
        b := a
        //c是切片，是引用类型，底层数组是a
        c := a[:]
        for i := 0; i < len(a); i++ {
          a[i] = a[i] + 1
        }
        //改变a的值后，b是a的拷贝，b不变，c是引用，c的值改变
        fmt.Println(a) //[2,3,4]
        fmt.Println(b) //[1 2 3]
        fmt.Println(c) //[2,3,4]

2.  make\
    make 只能用于slice, map 和 channel,
    所以下面一段代码生成了一个slice，是引用类型

        s1 := make([]int, 0, 3)

        for i := 0; i < cap(s1); i++ {
            s1 = append(s1, i)
        }
        s2 := s1
        for i := 0; i < len(a); i++ {
            s1[i] = s1[i] + 1
        }

        fmt.Println(s1)  //[1 2 3]
        fmt.Println(s2)  //[1 2 3]

3.  当对slice append 超出底层数组的界限时

        //n1是n2的底层数组
        n1 := [3]int{1, 2, 3}
        n2 := n1[0:3]
        fmt.Println("address of items in n1: ")
        for i := 0; i < len(n1); i++ {
            fmt.Printf("%p\n", &n1[i])
        }
        //address of items in n1:
        //0xc20801e160
        //0xc20801e168
        //0xc20801e170
        fmt.Println("address of items in n2: ")
        for i := 0; i < len(n2); i++ {
            fmt.Printf("%p\n", &n2[i])
        }
        //address of items in n2:
        //0xc20801e160
        //0xc20801e168
        //0xc20801e170

        //对n2执行append操作后，n2超出了底层数组n1的j
        n2 = append(n2, 1)
        fmt.Println("address of items in n1: ")
        for i := 0; i < len(n1); i++ {
            fmt.Printf("%p\n", &n1[i])
        }
        //address of items in n1:
        //0xc20801e160
        //0xc20801e168
        //0xc20801e170

        fmt.Println("address of items in n2: ")
        for i := 0; i < len(n2); i++ {
            fmt.Printf("%p\n", &n2[i])
        }
        //address of items in n2:
        //0xc20803a2d0
        //0xc20803a2d8
        //0xc20803a2e0
        //0xc20803a2e8

4.  引用“失效”\
    实现了删除slice最后一个item的函数

        func rmLast(a []int) {
            fmt.Printf("[rmlast] the address of a is %p", a)
            a = a[:len(a)-1]
            fmt.Printf("[rmlast] after remove, the address of a is %p", a)
        }

    调用此函数后，发现原来的slice并没有改变

        func main() {
            xyz := []int{1, 2, 3, 4, 5, 6, 7, 8, 9}
            fmt.Printf("[main] the address of xyz is %p\n", xyz)
            rmLast(xyz)
            fmt.Printf("[main] after remove, the address of xyz is %p\n", xyz)
            fmt.Printf("%v", xyz) //[1 2 3 4 5 6 7 8 9]
        }

    打印出来的结果如下：

        [main] the address of xyz is 0xc2080365f0
        [rmlast] the address of a is 0xc2080365f0
        [rmlast] after remove, the address of a is 0xc2080365f0
        [main] after remove, the address of xyz is 0xc2080365f0
        [1 2 3 4 5 6 7 8 9]

    这里直接打印了slice的指针值，因为slice是引用类型，所以指针值都是相同的，我们换成打印slice的地址看下

        func rmLast(a []int) {
            fmt.Printf("[rmlast] the address of a is %p", &a)
            a = a[:len(a)-1]
            fmt.Printf("[rmlast] after remove, the address of a is %p", &a)
        }
        func main() {
            xyz := []int{1, 2, 3, 4, 5, 6, 7, 8, 9}
            fmt.Printf("[main] the address of xyz is %p\n", &xyz)
            rmLast(xyz)
            fmt.Printf("[main] after remove, the address of xyz is %p\n", &xyz)
            fmt.Printf("%v", xyz) //[1 2 3 4 5 6 7 8 9]
        }

    结果：

        [main] the address of xyz is 0xc20801e1e0
        [rmlast] the address of a is 0xc20801e200
        [rmlast] after remove, the address of a is 0xc20801e200
        [main] after remove, the address of xyz is 0xc20801e1e0
        [1 2 3 4 5 6 7 8 9]

    这次可以看到slice作为函数参数传入函数时，实际上也是拷贝了一份slice，因为slice本身是个指针，所以从现象来看，slice是引用类型



