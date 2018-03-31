
---
date: 2016-12-31T11:33:38+08:00
title: "Go:ReadonlyVariable"
description: ""
disqus_identifier: 1485833618686974858
slug: "Go:-Readonly-Variable"
source: "https://segmentfault.com/a/1190000005071763"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

只读变量的缺失，应该算 Go 语言 “设计缺陷”。举例来说，默认以 error
实例来判断错误类别，但这些可导出全局变量实际可被外部修改，那么就存在隐性风险。

\

在实际开发中，有很多需设置访问权限的内存敏感数据，包括只读、只写，或不可操作等，好在可借助
syscall 实现。

使用示例：

\

当然，可以在此基础上实现更多功能，基本原理类似。对于敏感数据，还应增加如下功能：

> 1.  身份验证：用 runtime.Caller 验证调用堆栈，仅允许指定函数调用。
>
> 2.  内存锁定：用 syscall.Mlock
>     将数据锁定在物理内存页，禁止交换到硬盘。
>
最新动态，请扫码关注\


