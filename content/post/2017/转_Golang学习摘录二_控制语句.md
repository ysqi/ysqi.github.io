
---
date: 2017-02-24T08:31:55+08:00
title: "Golang学习摘录二:控制语句"
description: ""
disqus_identifier: 1487896315951254744
slug: "Golangxue-xi-zhai-lu-er-:kong-zhi-yu-gou"
source: "http://www.jianshu.com/p/1f6eb23994fc"
tags: 
- golang 
categories:
- 编程语言与开发
---

if语句
------

    i f x > 0 {   // {是强制的,且必须和if在同一行
     return y
    } else {
     return x
    }

if 和 switch 接受初始化语句,通常用于设置一个(局部)变量。

    if err := Chmod(0664); err != nil { //nil 与 C 的 NULL 类似
    fmt.Printf(err)  //err 的作用域被限定在 if 内
    return err
    }

goto语句
--------

用 goto 跳转到一定是当前函数内定义的标签

    func myfunc() { 
            i := 0
     Here:     // 这行的第一个词,以分号结束作为标签,标签名区分大小写
            println(i)
            i++
            goto Here   // 跳转
    }

for语句
-------

Go 的 for 循环有三种形式,只有其中的一种使用分号。

    for init; condition; post { } // 和C的for一样
    for condition {}              // 和while一样
    for {}                        // 死循环

    1、
    sum := 0
    for i:=0; i<10; i++ {
    sum+=i ←sum = sum + i 的简化写法 } 
    2、
    for i,j:=0, len(a)-1; i<j; i,j=i+1,j-1 {
    a[i], a[j] = a[j], a[i] //平行赋值 
    }

break和continue
---------------

利用 break 可以提前退出循环,break 终止当前的循环。

    for i:=0; i<10; i++ { 
      if i>5{
          break //终止这个循环,只打印 0 到 5
       }
       println(i)
    }

循环嵌套循环时,可以在 break 后指定标签。用标签决定哪个循环被终止:

    J: for j:=0; j<5; j++ {
        for i:=0; i<10; i++ {
           if i>5{ 
              break J // 现在种植的是j循环，而不是i的那个
            }
            println(i)
          } 
    }

利用 continue 让循环进入下一个迭代,而略过剩下的所有代码。下面循环打印了
0 到 5。

    for i:=0; i<10; i++ { 
        if i>5{
            continue // 跳过循环中所有的代码println(i)
        }
        println(i)
    }

range
-----

range可用于循环，支持slice、array、string、map和channel，range
是个迭代器,当被调用的时候,从它循环的内容中返回一个键值对。基于
不同的内容,range 返回不同的东西。

    // 遍历array
    list := []string {"a", "b", "c", "d", "e", "f"}
     for k, v := range list { 
      // k,v键值对：k为key，v为value
    }
    // 遍历字符串
    for pos, char := range "aΦx" {
    fmt.Printf("character '%c' starts at byte position %d\n",char, pos)
    }

switch语句
----------

Go 的 switch
非常灵活。表达式不必是常量或整数,执行的过程从上至下,直到找到匹
配项,而如果 switch 没有表达式,它会匹配 true 。这产生一种可能——使用
switch 编写 if-else-if-else 判断序列。

    func unhex(c byte) byte { 
      switch {
        case '0'<=c&&c<='9': return c - '0'
        case 'a'<=c&&c<='f': return c - 'a' + 10
        case 'A'<=c&&c<='F': return c - 'A' + 10
      }
      return 0
     }

它不会匹配失败后自动向下尝试,但是可以使用 fallthrough 使其这样做。没 有
fallthrough:

    switch i {
     case 0:  // 空的case体
    case 1: 
                f() // 当i==0时，f不会被调用！
    }

而这样:

    switch i { 
      case 0:  fallthrough
      case 1:
            f() // 当i==0时，f会被调用！ 
    }

用 default 可以指定当其他所有分支都不匹配的时候的行为。

    switch i { 
      case 0: 
      case 1: 
             f()
      default:
             g() //当i不等于0或1时调用
    }

分支可以使用逗号分隔的列表。

    func shouldEscape(c byte) bool { 
        switch c {
          case ' ', '?', '&', '=', '#', '+':    // ,或者 “or”都可以
                        return true
        }
      return false
    }

使用 switch 对字节数组进行比较的例子:

    func Compare(a, b []byte) int {
        for i:=0; i< len(a)&&i< len(b); i++ {
          switch {
            case a[i] > b[i]:
              return 1
            case a[i] < b[i]:
              return -1
            }
        }
          switch { 1
            case len(a) < len(b):
              return -1
            case len(a) > len(b):
              return 1 
           }
      return 0
    }

