
---
date: 2016-12-31T11:34:21+08:00
title: "【Go】Go语言学习笔记-1-简介"
description: ""
disqus_identifier: 1485833661083245730
slug: "【Go】Goyu-yan-xue-xi-bi-ji--1-jian-jie"
source: "https://segmentfault.com/a/1190000003701747"
tags: 
- golang 
categories:
- 编程语言与开发
---

争取在入职前把《学习Go语言》这个文档看完，把学习的笔记写在博客中，作为记录，方便以后查阅。

练习的代码都放在我自己的GitHub中，地址为：\
<https://github.com/poemqiong/GoExercises>

1. 包阅读方法
-------------

例如要查看unicode/utf8包的内容\
godoc unicode/utf8 | less

2. package和import
------------------

pakckage
main必须首先出现，然后是import，然后是其他所有内容。当Go程序在执行时，首先调用的函数是main.main().

3. 编译和运行
-------------

编译：go build test.go\
直接运行：go run test.go

4. 分号
-------

如果两个（或更多）语句放在一行书写，它们必须用分号（;）分割，一般情况下，不需要在行末加分号。

5. 变量
-------

Go的变量类型不是int a，而是a int。声明和赋值是两个过程。

1.  不同类型变量的赋值\
    (1)\
    var a int\
    var b bool\
    a = 15\
    b = false\
    (2)\
    var {

           a int
           b bool

    }\
    (3)

2.  := 15

-   := false

-   相同类型变量的赋值

a, b := 20, 16\
\_, b := 34,35

注意：

-   \_是一个特殊的变量名，任何赋给它的值都会被抛弃。

-   Go的编译器会对声明却未使用的变量报错。

-   混合使用不同类型的变量会引起编译器的错误。

6. 常量
-------

在Go中，常量在编译时被创建，只能是数字、字符串或者布尔值。

7. 字符串
---------

Go中，字符串赋值之后是不能修改的。如果要修改，需要先转换为rune数组（rune是int32的别名）。\
两个字符串相加应该写为\
s := "a" +

    "b"

而不能写做：\
s := "a"

-   "b"\
    因为Go编译器会自动在每一行末尾加上分号。

8. 运算符
---------

运算符的优先级，第一行最高，最后一行最低：

-   / % &lt;&lt; &gt;&gt; & &\^

-   = | \^\
    == != &lt; &lt;= &gt; &gt;=\
    &lt;-\
    &&\
    ||

需要注意的是：

-   &\^表示位清除

-   没有逻辑非！

-   Go不支持运算符重载

9. 控制结构
-----------

-   if else

-   switch

-   goto

-   for

-   break and continue

-   range（迭代器）

10. array
---------

array由\[n\]&lt;type&gt;定义，n标示array的长度，而&lt;type&gt;标示类型。赋值后，不能修改数组的大小，而且所有元素的类型相同。\
一维数组：\
a := \[...\]int{1,2,3}\
a := \[3\]int{1,2,3}\
二维数组：\
a := 3int {{1,2}, {3,4}, {5,6}}

11. slices
----------

slice与array类似，但是可以添加新的元素，slice是一个指向array的指针，是引用类型（引用类型使用make创建）。\
slice总是与一个固定长度的array成对出现。

sl := make(\[\]int, 10)

### append

s0 := \[\]int{0,0}\
s1 := append(s0, 2, 3, 4)

### copy

-   函数copy从源slice src复制元素到目标dst，并且返回复制的元素的个数。

-   源和目标可以重叠。

-   元素复制的数量是len(src)和len(dst)中的最小值。

12. map
-------

### 声明

-   map\[&lt;from type&gt;\]&lt;to type&gt;

-   当只需要声明一个map时，使用make的形式：mdays
    := make(map\[string\]int)

### 添加

mdays\["Jan"\] = 31

### 判断是否存在

v, ok := mdays\["Jan"\]\
若存在的话，ok为true

### 删除

delete(mdays, "Jan")

13. 小结
--------

由于第一章是简介，讲的内容比较杂，相信在后续的使用中会越来越熟悉这些细节。

