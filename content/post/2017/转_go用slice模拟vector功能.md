
---
date: 2017-02-18T11:13:46+08:00
title: "go用slice模拟vector功能"
description: ""
disqus_identifier: 1487387626521121318
slug: "go-yong-slicemo-ni-vectorgong-neng"
source: "http://blog.csdn.net/sydnash/article/details/54966734"
tags:
- grpc
categories:
- 编程语言与开发
---

appendVector
============

``` - 编程语言与开发
a = append(a, b...)
```

copy
====

``` - 编程语言与开发
b = append([]T(nil), a...)
```

``` - 编程语言与开发
b = make([]T, len(a))
copy(b, a)
```

cut删除一段范围i\~j
===================

``` - 编程语言与开发
copy(a[i:], a[j:])
for k, n := len(a) - j + i, len(a); k < n; k++ {
    a[k] = nil //or the zero value of T
}
a = a[:len(a) - j + i]
```

delete删除指定i
===============

``` - 编程语言与开发
copy(a[i:], a[i+1:]
a[len(a] - 1] = nil //or zero value of T
a = a[:len(a)-1]
```

expand 在i位置扩展j个位置出来
=============================

``` - 编程语言与开发
a = append(a[:i], append(make([]T, j), a[i:]...)...)
```

extend 在最后扩展j个位置
========================

``` - 编程语言与开发
a = append(a, make([]T, j)...)
```

insert 在i位置插入
==================

``` - 编程语言与开发
a = append(a[:i], append([]T{x}, a[i:]...)...)
```

这里创建了一个新的slice，然后拷贝a的后半截到新slice，在拷贝新的slice到a，这里两次拷贝。
\
下面方法一次拷贝

``` - 编程语言与开发
a = append(a, nil)//or zero value of T
copy(a[i+1:], a[i:])
a[i] = x
```

insertVector 插入vector b
=========================

``` - 编程语言与开发
a = append(a[:i], append(b, a[i:]...)...)
```

pop
===

``` - 编程语言与开发
x, a = a[len(a)-1], a[:len(a)-1]
```

push
====

``` - 编程语言与开发
a = append(a, x)
```

shift
=====

``` - 编程语言与开发
x, a := a[0], a[i:]
```

unshift
=======

``` - 编程语言与开发
a = append([]T{x}, a...)
```

filter
======

``` - 编程语言与开发
b := a[:0]
for _, x := range a {
    if f(x) {
        b = append(b, x)
    }
}
```

reversing
=========

``` - 编程语言与开发
for left, right := 0, len(a) - 1; left < right; left, right = left - 1, right - 1 {
    a[left], a[right] = a[right], a[left]
}
```

