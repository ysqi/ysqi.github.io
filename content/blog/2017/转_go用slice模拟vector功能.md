
---
date: 2017-02-18T11:13:46+08:00
title: "go用slice模拟vector功能"
description: ""
disqus_identifier: 1487387626521121318
slug: "go-yong-slicemo-ni-vectorgong-neng"
source: "http://blog.csdn.net/sydnash/article/details/54966734"
tags: 
- golang#grpc 
topics:
- 编程语言与开发
---

appendVector
============

``` {.prettyprint}
a = append(a, b...)
```

copy
====

``` {.prettyprint}
b = append([]T(nil), a...)
```

``` {.prettyprint}
b = make([]T, len(a))
copy(b, a)
```

cut删除一段范围i\~j
===================

``` {.prettyprint}
copy(a[i:], a[j:])
for k, n := len(a) - j + i, len(a); k < n; k++ {
    a[k] = nil //or the zero value of T
}
a = a[:len(a) - j + i]
```

delete删除指定i
===============

``` {.prettyprint}
copy(a[i:], a[i+1:]
a[len(a] - 1] = nil //or zero value of T
a = a[:len(a)-1]
```

expand 在i位置扩展j个位置出来
=============================

``` {.prettyprint}
a = append(a[:i], append(make([]T, j), a[i:]...)...)
```

extend 在最后扩展j个位置
========================

``` {.prettyprint}
a = append(a, make([]T, j)...)
```

insert 在i位置插入
==================

``` {.prettyprint}
a = append(a[:i], append([]T{x}, a[i:]...)...)
```

这里创建了一个新的slice，然后拷贝a的后半截到新slice，在拷贝新的slice到a，这里两次拷贝。
\
下面方法一次拷贝

``` {.prettyprint}
a = append(a, nil)//or zero value of T
copy(a[i+1:], a[i:])
a[i] = x
```

insertVector 插入vector b
=========================

``` {.prettyprint}
a = append(a[:i], append(b, a[i:]...)...)
```

pop
===

``` {.prettyprint}
x, a = a[len(a)-1], a[:len(a)-1]
```

push
====

``` {.prettyprint}
a = append(a, x)
```

shift
=====

``` {.prettyprint}
x, a := a[0], a[i:]
```

unshift
=======

``` {.prettyprint}
a = append([]T{x}, a...)
```

filter
======

``` {.prettyprint}
b := a[:0]
for _, x := range a {
    if f(x) {
        b = append(b, x)
    }
}
```

reversing
=========

``` {.prettyprint}
for left, right := 0, len(a) - 1; left < right; left, right = left - 1, right - 1 {
    a[left], a[right] = a[right], a[left]
}
```

