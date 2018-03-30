
---
date: 2016-12-31T11:34:40+08:00
title: "go(golang)之slice的小想法1"
description: ""
disqus_identifier: 1485833680451040648
slug: "go(golang)zhi-slicede-xiao-xiang-fa-1"
source: "https://segmentfault.com/a/1190000000718348"
tags: 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

slice，是go中一个很重要的主题。我们不用切片来表述，因为这里的切片特指的是数组的切片。

先给slice下个定义吧：
---------------------

> Slice expressions construct a substring or slice from a string, array,
> pointer to array, or slice. There are two variants: a simple form that
> specifies a low and high bound, and a full form that also specifies a
> bound on the capacity.

从一个字符串中构建了一个子字符串或者从一个数组中构建一个切片，并且把这个子字符串或是这个切片的指针赋给这个slice.换句话说slice就是指向某个字符串或者某个数组的一个指针。

表面上来看，slice是一种与array很相似的东西，但是两者之间最大的区别是array是定长的而slice可以更改其长度。

slice的基本语法：
-----------------

### 1

    a[low : high]

1.  创建一个数组

    a := \[5\]int{1, 2, 3, 4, 5}

2.  创建切片

    s := a\[1:4\]

3.  slice
    s中元素的类型是int,length(长度)是3（high-low）,capacity（容量）是4

### 2 切片的省略写法

> （原文呈现）For convenience, any of the indices may be omitted. A
> missing low index defaults to zero; a missing high index defaults to
> the length of the sliced
> operand。（为了方便，任何索引都是可以被忽略的，开始的索引默认为0，结束的索引默认为slice的长度）

    a[2:]  // same as a[2 : len(a)]
    a[:3]  // same as a[0 : 3]
    a[:]   // same as a[0 : len(a)]

### 3

    slicer := make([]int, 10)

可以通过make来新建一个slice,第一个参数是slice中的元素类型，第二个参数是这个slice的容量。

slice的实现原理
---------------

当我们创建了一个slice的时候会发生以下的事情：

1.  创建了一个与参数1相匹配元素类型的、参数2长度的数组。

2.  创建指向这个数组的指针并赋给这个切片。

这里的指针其实就是指向了slice索引值start值对应的数组元素的位置的地址。

`func main() {     a := [5]int{1, 2, 3, 4, 5}     var d *[5]int = &a     println(d)     println(a[0:4])     println(a[1:4])}`

**结果：**

         0x220822bf08//原始数组地址
    [4/5]0x220822bf08//对应切片a[0:4]地址
    [3/4]0x220822bf10//对应切片a[1:4]地址

