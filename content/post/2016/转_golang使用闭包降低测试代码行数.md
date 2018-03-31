
---
date: 2016-12-31T11:32:56+08:00
title: "golang使用闭包降低测试代码行数"
description: ""
disqus_identifier: 1485833576803450555
slug: "golangshi-yong-bi-bao-jiang-di-ce-shi-dai-ma-hang-shu"
source: "https://segmentfault.com/a/1190000007300284"
tags: 
- 闭包 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

有如下函数，简单来说就是有错误则直接返回，没错误则执行`f`函数。

    func (t *transaction) Do(f func()) *transaction {
        if t.fail || t.rollback || t.finish {
            return t
        }
        f()
        return t
    }

函数很简单，但如何测试呢，简单但丑陋的方法：

    func Test_func(t *testing.T) {
        isCalled := false
        f := func() {
            isCalled = true
        }
        trans := New()
        // do something
        trans.Do(f)
        // check
        if isCalled {
            // do something
        }
    }

在`f`中修改外部变量，然后判断变量是否变化就可以知道`f`是否被执行。但我们一般需要测试多种情况，比如对于`Do`函数，我们需要将
`t.fail` `t.rollback`
`t.finish`设置不同值进行测试，将上面测试代码扩充（如果需要测试这三个变量组合的情况，代码就更长了）：

    func Test_func(t *testing.T) {
        isCalled1 := false
        isCalled2 := false
        isCalled3 := false
        f1 := func() {
            isCalled1 = true
        }
        f2 := func() {
            isCalled2 = true
        }
        f3 := func() {
            isCalled3 = true
        }
        trans := New()
        // do something
        trans.Do(f1)
        trans.Do(f2)
        trans.Do(f3)
        // check
        if isCalled1 {
            // do something
        }
        if isCalled2 {
            // do something
        }
        if isCalled3 {
            // do something
        }
    }

在上面代码中`f1` `f2`
`f3`函数的逻辑都一样，这时可以通过使用闭包来消除冗余代码：

    func Test_func(t *testing.T) {
        genTestFunc := func() (func(), func() bool) {
            isCalled := false
            return func() {
                    isCalled = true
                }, func() bool {
                    return isCalled
                }
        }
        f1, f1Called := genTestFunc()
        f2, f2Called := genTestFunc()
        f3, f3Called := genTestFunc()
        trans := New()
        // do something
        trans.Do(f1)
        trans.Do(f2)
        trans.Do(f3)
        // check
        if f1Called() {
            // do something
        }
        if f2Called() {
            // do something
        }
        if f3Called() {
            // do something
        }
    }

解释一下，`genTestFunc `返回值是两个函数，第一个函数可传入`Do`中，第二个函数用来判断是否被`Do`调用。\
粗略看改动前后代码行数基本相同，但如果`f`变复杂或者需要更多的测试case时，改动后的代码更加简洁，易于维护。

