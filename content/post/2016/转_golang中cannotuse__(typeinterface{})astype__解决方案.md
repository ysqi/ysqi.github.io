
---
date: 2016-12-31T11:34:04+08:00
title: "golang中cannotuse**(typeinterface{})astype**解决方案"
description: ""
disqus_identifier: 1485833644379465814
slug: "golang-zhong--cannot-use-**-(type-interface-{})-as-type-**jie-jue-fang-an"
source: "https://segmentfault.com/a/1190000004112321"
tags: 
- interface 
- beego 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

在beego中从session中取值的时候，取出来的是intergace{}，但是我先返回的值是int型，或者是string，这个时候会出现一个错误：\
`cannot use ** （变量）(type interface {}) as type **（类型）`

错误代码：

    func CurrentId(ctx *context.Context) int {
        userStr := ctx.Input.CruSession.Get("user_id")
        return userStr
    }

从session中取出来的是一个`interface`类型，无法直接转换，我在给user\_id赋值的时候是给的int类型。\
因此直接对userStr进行转换即可。

正确代码如下：

    func CurrentId(ctx *context.Context) int {
        userStr := ctx.Input.CruSession.Get("user_id")
        return userStr.(int)
    }

