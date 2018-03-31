
---
date: 2016-12-31T11:33:09+08:00
title: "golang什么时候应该把方法绑定在struct的值上而不是指针上？"
description: ""
disqus_identifier: 1485833589191753555
slug: "golangshen-me-shi-hou-ying-gai-ba-fang-fa-bang-ding-zai-structde-zhi-shang-er-bu-shi-zhi-zhen-shang-"
source: "https://segmentfault.com/a/1190000006803598"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

golang 支持 struct 也支持 struct 的指针。一个常见的困惑是既然struct
指针存在了，为什么不干脆只有struct的指针呢？两个原因：

-   struct不可空，而struct指针可以为nil

-   \[\]my\_struct的内存是连续的，而\[\]\*my\_struct只有指针是连续存放的，而实际的内容则需要跟随指针去读取

同时struct应该也有助于escape
analysis，使得对象可以分配在栈上，而不用去进行多线程争抢的分配堆上的内存。但是这只是一个猜想。

一个更有趣的问题是，什么时候方法应该绑定在struct上，而不是struct指针上？考虑以下测试程序

    type MyStruct struct {
        field int
    }

    local_variable := MyStruct{1}
    fmt.Println(unsafe.Pointer(&local_variable))
    local_variable.modify_struct_value()
    fmt.Println(local_variable) // {1}
    copied := local_variable.copy_my_self()
    fmt.Println(unsafe.Pointer(&copied))

    func (self MyStruct) modify_struct_value() {
        fmt.Println(unsafe.Pointer(&self))
        self.field = 2
    }
    func (self MyStruct) copy_my_self() MyStruct {
        fmt.Println(unsafe.Pointer(&self))
        return self
    }

执行的输出是

    0xc82000a2c8
    0xc82000a2e0
    {1}
    0xc82002a028
    0xc82000a2f0

这个证明了几个事情

-   在方法执行之前struct就被拷贝了一份

-   对这份拷贝的修改，不会影响原struct（废话）

-   如果返回这份拷贝给调用方，调用方在保存在局部变量的时候还会再拷贝一遍（没有return
    value optimization）

根据这样的行为，除非我们想让一个方法是类似c++的const的行为（以内存拷贝为代价），其他的场景下基本没有理由把行为绑定在struct上而不是绑定在struct指针上。

结论：

-   struct因为可以控制内存layout，有其存在意义。性能不敏感的场景，一直使用struct指针也无需有负罪感。

-   总是把方法绑定在struct指针上，而不要绑定在struct上，以避免不必要的内存拷贝



