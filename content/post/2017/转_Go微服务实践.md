
---
date: 2017-02-22T08:37:14+08:00
title: "Go微服务实践"
description: ""
disqus_identifier: 1487723834797940463
slug: "Go-wei-fu-wu-shi-jian"
source: "https://my.oschina.net/ifraincoat/blog/842021"
tags: 
- golang#微服务 
categories:
- 编程语言与开发
---

简介
----

 

近一两年来，微服务架构已经成为热门话题（microservices.io)，与传统的一体化应用架构相比，微服务架构在开发、测试、部署方面都有众多吸引人之处，越来越多没有历史包袱的新项目都启用微服务架构的模式来开发。

我们这个团队经过深入思考之后，决定在一起美这个APP的后端开发中，选择Go作为开发语言，采用微服务模式来实现，经过近半年的实践，形成了一些心得，简单总结后分享出来，希望能够给大家一些帮助。

框架选择
--------

不同的团队在选择基础框架（库）时考虑的要素不同，我们团队更喜欢小而美的框架，尽可能不要让框架侵入业务，易于升级、维护和替换，所以我们更愿意选择Library而不是Framework。

在web方面，我们选择了negroni作为middleware库，采用性能不错的httprouter替换go标准库的mux，而没有用任何web相关的框架。

在微服务之间的rpc调用方面，为了将来的扩展性、跨语言调用等因素，我们没有直接用go标准库的rpc模块，而是采纳了google最新推出的grpc。但grpc本身属于比较重型的rpc框架，对业务代码有一定的侵入性，我们做了一系列的库（包括worpc、worc、wonaming等https://github.com/wothing）来屏蔽这些不必要的业务代码侵入，保持了业务代码本身的整洁。

微服务划分
----------

在微服务体系中，如何切分微服务也是一个重要的话题，在我们的实践中，我们遵循了如下一些原则：

1.  逻辑独立、边界清晰的模块作为一个独立的微服务

2.  每个table只由一个微服务操作（包括插入、读取、更改、删除等）

3.  table之间不引入外键约束，id字段全部采用uuid

4.  将需要保持数据一致性的操作放在一个微服务中，避免跨服务带来的数据一致性难题

5.  微服务之间的通信，尽可能采用消息队列实现松耦合，当需要同步调用时再借助于rpc

6.  微服务独立部署，通过etcd实现服务的注册与发现

总体架构
--------

![](https://static.yushuangqi.com/blog/2017/0222083427akeh45ct1ujjpg.jpg)

> Gateway
> -------

Gateway是微服务对外提供服务的一个屏障，它的核心点在于：

1.  屏蔽微服务之间通过消息队列、rpc等通信方式，为Web页面和移动APP提供基于HTTP协议的RESTful
    API接口

2.  对每一个http业务请求进行必要的鉴权和数据完整性、合法性检查，以减少微服务的负担，让微服务的代码更纯粹

3.  微服务部署体系中，每个微服务可能会部署多个实例，Gateway还承担着在这些实例中进行负载均衡的功能

4.  进行必要的日志输出、监控打点等功能，对每一个来自于APP和页面的http请求，生成一个唯一的trace
    id，并将trace id传导到每一个后续的微服务中，以便后续的查错和性能调优

5.  Gateway的每一个http请求都是无状态的，采用JWT（Json Web
    Token）机制实现一个客户端的请求状态信息的传递

> 服务的注册与发现wonaming
> ------------------------

微服务体系中，服务的注册和发现对整体架构非常重要，尤其对于同步的rpc调用，每个服务有多少实例，每个实例的地址等，都需要有一个统一的管理。我们采用etcd保存服务信息，同时封装了wonaming作为微服务注册和发现的中间件，它的主要功能包括：

1.  服务在启动时，调用wonaming向etcd注册包含TTL的服务“索引”、

2.  注册后，服务与etcd保持定时心跳，当微服务主动退出或超时，服务解注册并“下线”

3.  在Gateway中，通过resolver进行服务发现，配合grpc提供的balancer实现负载均衡，resolver启动后会对etcd中的 /wonaming 目录进行监控，当有服务注册或者解注册时，动态维护可用服务清单。

>      
>
> r := wonaming.NewResolver(name) b := grpc.RoundRobin(r) conn, err :=
> grpc.Dial(etcd, grpc.WithInsecure(), grpc.WithBalancer(b))

> 服务的rpc调用worc
> -----------------

grpc是一个比较重的rpc框架，当客户端通过grpc调用服务端时，需要大量的重复性代码来建立连接、调用、处理错误返回等，影响业务代码的整洁性，并且对业务代码具有很强的侵入性，为了规避这个问题，我们封装了worc，以实现便捷的grpc调用：

>      
>
> resp, err := worc.CallRPC(ctx, "hello", "Hello", req)

> grpc的中间件链worpc
> -------------------

grpc提供了interceptor机制，但并没有提供chain来实现不同的中间件的顺序执行，为了将不同的中间件功能（如鉴权、日志、recover）封装在不同的函数里，worpc提供了组合gprc
interceptor为一个chain的能力，可以根据自身业务的需要，撰写不同的grpc中间件进行组合，比如实现
grpc 的 recovery 与 log 中间件：

>      
>
> func Recovery(ctx context.Context, req interface{}, info
> \*grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{},
> err error) { defer func() { if r := recover(); r != nil { // log stack
> stack := make([]byte, MAXSTACKSIZE) stack =
> stack[:runtime.Stack(stack, false)] log.CtxErrorf(ctx, "panic grpc
> invoke: %s, err=%v, stack:\\n%s", info.FullMethod, r, string(stack))
> // if panic, set custom error to 'err', in order that client and sense
> it. err = grpc.Errorf(codes.Internal, "panic error: %v", r) } }()
> return handler(ctx, req) } func Logging(ctx context.Context, req
> interface{}, info \*grpc.UnaryServerInfo, handler grpc.UnaryHandler)
> (resp interface{}, err error) { start := time.Now() log.CtxInfof(ctx,
> "calling %s, req=%s", info.FullMethod, marshal(req)) resp, err =
> handler(ctx, req) log.CtxInfof(ctx, "finished %s, took=%v, resp=%v,
> err=%v", info.FullMethod, time.Since(start), marshal(resp), err)
> return resp, err }

>      
>
> s :=
> grpc.NewServer(grpc.UnaryInterceptor(worpc.UnaryInterceptorChain(worpc.Recovery,
> worpc.Logging)))

通过以上组合，可为微服务提供panic恢复能力，保障服务稳定可用；同时还将上文中提到的注入context中的trace
id取出，这样Gateway与微服务的日志通过trace
id就衔接了起来，方便查错、调优等。

> 其它经验
> --------

1.  使用grpc.Errorf封装业务中的逻辑错误，随grpc服务调用一起返回，将业务response与error
    分离。

2.  数据可在Gateway中完成组装工作，但无需刻意避免微服务互调，理清依赖关系，尤其当protobuf升级时，根据具体业务来判断引用微服务是否需要同步重部署。

3.  微服务虽好，但一定程度上会加大实施难度，要根据业务体量合理入坑。

总结
----

以上是微服务架构在我们团队的实践方案，麻雀虽小，五脏俱全。通过各中间件的灵活组合，保障业务有序与服务的高可用，还不抓紧实践起来？在后续的文章中，我们还会介绍目前微服务测试、运维及部署方案。

 

本文作者：一起美 - Elvizlai  

github：https://github.com/elvizlai

