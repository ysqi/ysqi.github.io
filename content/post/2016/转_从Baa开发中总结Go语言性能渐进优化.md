
---
date: 2016-12-31T11:33:28+08:00
title: "从Baa开发中总结Go语言性能渐进优化"
description: ""
disqus_identifier: 1485833608762791537
slug: "cong-Baakai-fa-zhong-zong-jie-Goyu-yan-xing-neng-jian-jin-you-hua"
source: "https://segmentfault.com/a/1190000005690821"
tags: 
- baa 
- gc 
- golang 
categories:
- 编程语言与开发
---

在Go生态已经有很多WEB框架，但感觉没有一个符合我们的想法，我们想要一个`简洁高效`的核心框架，提供`路由`，`context`，`中间件`和`依赖注入`，而且拒绝使用`正则`和`反射`，于是我们开始构建[Baa](https://github.com/go-baa/baa)框架。一开始使用最简单的通俗写法实现了第一版的功能，基本可用，但是性能烂到爆，优化之路漫漫开启。

> 最好的文章应该是每一步都加上优化前后的benchmark对比结果，给读者以最直观的感受。我先BS一下自己，因为我懒了，没有再回头一步步去对比这个结果图。

### 拒绝正则和反射

这是我们做这个框架时的一个基本原则，整个实现中没有使用过regexp、reflect包。这是我们对性能追求的基础。带来的另一个收益是，没有魔法，都是非常容易理解的实现，让整个框架变得简单。

### 使用sync.Pool重用对象

在我上次翻译的文章[CockroachDB
GC优化总结](https://segmentfault.com/a/1190000004046212)中介绍过这些方法，在《Go语言圣经》中作者也介绍了这个方法，使用
sync.Pool
可以在一次GC之间重用对象，避免对象的频繁创建和内存分配。我们在追求性能的过程中，要尽可能减少甚至达到内存零分配，这是一个最重要的用法。

在Baa中有如下代码片段：

    b.pool = sync.Pool{
        New: func() interface{} {
            return newContext(nil, nil, b)
        },
    }

使用的时候：

    c := b.pool.Get().(*Context)
    c.reset(w, r)

使用完：

    b.pool.Put(c)

### 使用array优化slice

slice的本质就是就是一个可变长度的array，根据存储的容量会动态的重新分配内存迁移数据。如果长度不断变化，会导致不断的重新分配内存，在特定场景下，如果我们可以使用一个定长的array来优化内存分配。

    var nameArr [1024]string
    pNames := nameArr[0:0]
    pNames = append(pNames, "val")

pNames 是一个slice，但数据操作总是在array
nameArr上完成，在整个使用过程中不会重新分配内存。

> 上面的伪代码，在Baa中已经不存在了，Baa改用了下面的技巧来取代定长的array。

### slice也能重用

slice的重用，其实和上面的利用array优化基本一致，就是初始分配一个较大的容量，尽可能在使用的过程中都不会超出容量，当然也不用担心，万一不够用了，会自动扩容，只不过会进行一次内存分配。

在Baa中有如下代码片段：

    // newContext create a http context
    func newContext(w http.ResponseWriter, r *http.Request, b *Baa) *Context {
        c := new(Context)
        c.Resp = NewResponse(w, b)
        c.baa = b
        c.pNames = make([]string, 0, 32)
        c.pValues = make([]string, 0, 32)
        c.handlers = make([]HandlerFunc, len(b.middleware), len(b.middleware)+3)
        copy(c.handlers, b.middleware)
        c.reset(w, r)
        return c
    }

    // reset ...
    func (c *Context) reset(w http.ResponseWriter, r *http.Request) {
        c.Resp.reset(w)
        c.Req = r
        c.hi = 0
        c.handlers = c.handlers[:len(c.baa.middleware)]
        c.pNames = c.pNames[:0]
        c.pValues = c.pValues[:0]
        c.store = nil
    }

注意newContext中的 c.pNames和c.pValues 以及 reset中的
c.pNames和c.pValues，通过 slice\[:0\]
来重用之前的slice，避免内存重新分配。至于上面的长度32，是根据经验得来的一个值，尽可能保证长度满足大部分情况下的需求又不太大。

### 使用Radix tree重写路由

之前在`黑夜路人微信群`中还讨论过一个问题：算法、数据结构，在实际工作中有用到过吗？说实话，一般情况下真不怎么用到，不过这里就是一个场景。

在第一版中，路由就是一个map，路由匹配就是一个range，简单，清晰，但性能自然不好。参考了
`macaron`和`echo`框架的设计，都是使用`基数树(radix tree)`来实现的，只是实现的细节不同，这里我们也有不同的细节实现，但思路基本没变。具体实现可以参考
[wiki](https://en.wikipedia.org/wiki/Radix_tree)，和 Baa router部分
[router.go](https://github.com/go-baa/baa/blob/master/router.go)

### string的性能不怎样

很多文章介绍过了，尽量使用 \[\]byte 替代 string，这里我们也是这么做的。

### Map的range好低效

map和slice的range性能差一个数量级啊，所以，你会发现我们取消了大量的map改为了slice，在`slice也能重用`这一节的代码示例中
pNames和pValues就是用来取代原来的 map\[string\]string，因为map
range的效率太低了。

### 凡是迭代就有开销

slice的迭代是很快，可是总还是迭代，是迭代就有开销，为了追求极致的性能也是疯了。在路由匹配时，我们给所有的路由pattern设置了单字节的index，如果首字母都不匹配，就没有必要继续后面的字符匹配了。

路由条目创建：

    // newRoute create a route item
    func newRoute(pattern string, handles []HandlerFunc, router *Router) *Route {
        r := new(Route)
        r.pattern = pattern
        r.alpha = pattern[0]
        r.handlers = handles
        r.router = router
        r.children = make([]*Route, 0)
        return r
    }

路由条目匹配：

    // findChild find child static route
    func (r *Route) findChild(b byte) *Route {
        var i int
        var l = len(r.children)
        for ; i < l; i++ {
            if r.children[i].alpha == b && !r.children[i].hasParam {
                return r.children[i]
            }
        }
        return nil
    }

注意 `r.alpha` 就是用来尽可能避免迭代进一步提高性能的。

### defer也仅是方便

在追求极致性能的路上，我都快疯了，在一步步测试的过程中，发现去掉defer也能提高一些性能，`雨痕学堂`微信公众号
中的一篇文章也提到了这个问题，因为defer有额外的开销来保证延迟调用甚至panic时也能执行，而大多数时候我们可以在程序的结束时直接终止，避免defer机制，再快一点点。

### 函数调用也是开销

离目标越来越近，但还有一点差距，我们也越来越疯狂，最后居然干成了这样，我们把部分频繁调用的函数取消，改为直接在一个函数中完成，因为我们发现，即使只是一个函数调用，TMD也是开销呀。

### pprof是神器

在整个过程中，如何一步步分析性能问题，定位可优化的地方，go test
-cpuprofile, go test -memprofile, go test -bench
就是最好的工具，每修改一次，bench看结果，profile看性能分析。

### 总结

本文简单总结了在优化过程中的各种技巧，和部分代码示例，更多使用姿势，自行体验，欢迎交流和拍砖。

