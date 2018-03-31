
---
date: 2016-12-31T11:34:05+08:00
title: "[译]CockroachDBGC优化总结"
description: ""
disqus_identifier: 1485833645054799372
slug: "[yi-]-CockroachDB-GCyou-hua-zong-jie"
source: "https://segmentfault.com/a/1190000004046212"
tags: 
- cockroachdb 
- gc 
- golang 
categories:
- 编程语言与开发
---

几周前我们分享了一个帖子讲述我们为什么选择Go语言编写CockroachDB，我们收到一些问题，询问我们是如何解决Go语言的一些已知问题，特别是关于性能、GC和死锁的问题。

本文中我们将分享几个非常有用的优化技巧用以改善许多常见的GC性能问题（接下来还将覆盖一些有趣的死锁问题）。我们将重点分享如何通过嵌套结构体、使用
sync.Pool、和复用后端数组减少内存分配和降低GC开销。

### **减少内存分配和GC优化**

将Go与其他语言（比如java）区别开来的是Go语言能让你管理内存布局。通过GO语言，你可以合并碎片，而其他垃圾集合语言不能。

让我们看看CockroachDB中从磁盘读取数据并解码的一小段代码：

    metaKey := mvccEncodeMetaKey(key)
    var meta MVCCMetadata
    if err := db.GetProto(metaKey, &meta); err != nil {
        // Handle err
    }
    ...
    valueKey := makeEncodeValueKey(meta)
    var value MVCCValue
    if err := db.GetProto(valueKey, &value); err != nil {
        // Handle err
    }

为了读取数据，我们执行了4次内存分配：MVCCMetadata结构体、MVCCValue结构体和metaKey、valueKey。在Go语言中我们可以通过合并结构体和预分配空间给Key把内存分配减少为1次。

    type getBuffer struct {
        meta MVCCMetadata
        value MVCCValue
        key [1024]byte
    }

    var buf getBuffer
    metaKey := mvccEncodeKey(buf.key[:0], key)
    if err := db.GetProto(metaKey, &buf.meta); err != nil {
        // Handle err
    }
    ...
    valueKey := makeEncodeValueKey(buf.key[:0], meta)
    if err := db.GetProto(valueKey, &buf.value); err != nil {
        // Handle err
    }

我们声明了一个getBuffer类型，包含两个不同的结构体：MVCCMetadata和MVCCValue（都是protobuf对象），不同于通常使用的切片，第三个成员使用了一个数组。

不需要额外分配内存，你就可以直接在结构体中定义一个定长的数组（1024
bytes），这允许我们将三个对象放到同一个getBuffer结构体中。这样我们就把4次内存分配减少为1次。需要注意的的两个不同的key我们使用了同一个数组，在两个key不同时使用的情况下是可以正常工作的。稍后我们再来讨论数组。

### **sync.Pool**

    var getBufferPool = sync.Pool{
           New: func () interface{} {
                  return &getBuffer{}
           },
    }

说实话，我们花了一段时间才弄明白为什么 sync.Pool
才是我们我们想要的。在一个GC周期内可以无限制使用同一个对象无需多次内存分配，GC会负责回收。在每次GC启动的时候都会清除Pool中的对象。

用一个例子来说明如何使用 sync.Pool:

    buf := getBufferPool.Get().(*getBuffer)
    defer getBufferPoolPut(buf)

    key := append(but.key[0:0], ...)

首先你需要使用一个工厂函数来声明一个全局的 sync.Pool
对象，在这个列子中我们分配一个 getBuffer结构体并返回。我们不再创建新的
getBuffer 改为从 pool 中获取。Pool.Get
返回的是一个空接口，我们需要使用类型断言转换。使用完成后再放回到 pool
中。最终的结果是我们无需每次获取 getBuffer时都分配一次内存。

### **数组和切片**

有些事可能不值一提，在Go语言中数组和切片是不同的类型，而且切片和数组几乎所有操作都一样。你仅仅通过一个方括号语法
\[:0\] 就可以从数组得到一个切片。

    key := append(bf.key[0:0], ...)

这里使用数组创建了一个长度为0的切片。事实是这个切片已经拥有了一个后端存储，意思是说对切片的append操作实际上插入到数组中，而并没有分配新的内存。所以当我们解码一个key时，我们可以append进一个通过这个
buffer 创建的切片中。只要key的长度小于 1
KB，我们就不需要做任何内存分配。将复用我们给数组分配的内存。

key 的长度超过 1 KB
的情况可能会有但是不常见，在这种情况下，程序可以透明的自动分配新的后端数组，我们的代码不需要做任何处理。

### **Gogoprotobuf vs Google protobuf**

最后，我们在磁盘上存储所有的数据都使用了protobuf。然而我们并没有使用
Google官方的protobuf类库，我们强烈推荐使用一个叫做 gogoprotobuf的分支。

Gogoprotobuf
遵循了很多我们上面提到的关于避免不必要的内存分配的原则。尤其是，它允许将数据编码到一个后端使用数组的字节切片以避免多次内存分配。此外，非空注解允许你直接嵌入消息而无需额外的内存分配开销，这在始终需要嵌入消息时是非常有用的。

最后一点优化是，较基于反射进行编码和解编码的Google标准protobuf类库，gogoprotobuf使用编码和解编码协程提供了不错的性能改善。

### **总结**

通过结合上述技巧，我们已经可以最小化GC的性能开销和优化更好的性能。当我们接近测试阶段，更多地专注于内存分析，我们将在后续的帖子中分享我们的成果。当然，如果你知道其他的Go语言性能优化，我们洗耳恭听。

> 原文链接：[](http://www.cockroachlabs.com/blog/how-to-optimize-garbage-collection-in-go/)<http://www.cockroachlabs.com/blog/how-to-optimize-garbage-collection-in-go/>\
> 原文作者：Jessica Edwards\
> 翻译校对：betty, 龙猫，柚子

