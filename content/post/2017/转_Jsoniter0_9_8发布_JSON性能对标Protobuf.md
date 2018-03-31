
---
date: 2017-02-17T08:17:15+08:00
title: "Jsoniter0_9_8发布:JSON性能对标Protobuf"
description: ""
disqus_identifier: 1487290635473620447
slug: "Jsoniter-0_9_8-fa-bu-:-JSON-xing-neng-dui-biao--Protobuf"
source: "https://segmentfault.com/a/1190000008295838"
tags: 
- protobuf 
- json 
- golang 
- java 
categories:
- 编程语言与开发
---

Jsoniter 是一款快且灵活的 JSON 解析器，同时提供
[Java](https://github.com/json-iterator/java) 和
[Go](https://github.com/json-iterator/go) 两个版本。

最近发布的 0.9.8 版本对性能对标 Jackson 和 Protobuf 进行了详细的评测：
[https://github.com/json-itera...](https://github.com/json-iterator/java-benchmark)
。性能优化的原理会近期会发布于 infoq 中文站，尽请期待。

同时提供 PHP 一般的体验。在 PHP 里，你只需要记得 json\_decode
，什么文档都可以解析。现在在 Java 里，你也可以这么做了。

    Any any = Jsoniter.deserialize(input); // deserialize 返回 "Any"，实际的解析是延迟在读取时才做的
    any.get("items", '*', "name", 0); // 抽取所有 items 的第一个 name
    any.get("size").toLong(); // 不管是 "100" 还是 100 ，都给转成 long 类型，就像弱类型一样
    any.bindTo(Order.class); // 把 JSON 绑定到对象
    for (Any element : any) {} // 遍历集合， Any 实现了 iterable 接口

项目网站：
[http://jsoniter.com/index.cn....](http://jsoniter.com/index.cn.html)

