
---
date: 2017-03-03T08:36:23+08:00
title: "为什么Go语言在中国格外的火"
description: ""
disqus_identifier: 1488501383780881405
slug: "wei-shen-me-Goyu-yan-zai-zhong-guo-ge-wai-de-huo-&amp"
source: "http://blog.csdn.net/wangshubo1989/article/details/55102275"
tags: 
- golang
topics:
- 编程语言与开发
---

go语言推出有几年了，似乎不温不火。但是在中国范围内，确实被关注的一塌糊涂。

这是2017年2月份TIOBE出的编程语言排名：

![编程语言排名](https://static.yushuangqi.com/blog/2017/0303081604yyeghltmjxt.png)

在拉勾网上搜索go的职位，结果有119个(2017年2月14日搜索结果)，似乎还没有那么火爆：

![拉勾网上搜索go的职位](https://static.yushuangqi.com/blog/2017/03030816043ar4ljq1c11.png)

但是在中国，很多公司，很多程序员都在谈论go语言，也就是说在中国对于go的关注异常火爆。

根据谷歌搜索的统计，如下图： 
 The graph above shows the searches for “golang” by country on Google
Trends. 

![Go谷歌搜索](https://static.yushuangqi.com/blog/2017/0303081604i2dl1kq4wh1.png)

外国人专门写了一篇文章，来分析为什么go在中国如此火： 
( 《Why is Golang popular in China?》)[http://herman.asia/why-is-go-popular-in-china](http://herman.asia/why-is-go-popular-in-china)

下面是知乎的回复： 
 作者：匿名用户 

链接：[https://www.zhihu.com/question/30172794/answer/47122000](https://www.zhihu.com/question/30172794/answer/47122000)

 来源：知乎 
 著作权归作者所有，转载请联系作者获得授权。


这个“火”字看你怎么理解了。
Go在国内更火只是感觉上的。比如推文，以及谈论的相关话题较多而已(但能有nodejs多么？)，本身中国人口数量就多，按这个衡量的办法去看的话，swift在国内也比在国外火；
实际上Go在国外更火（这里的火是实际的使用情况），对比一下国内和国外使用Go的程度、数量，Go相关的技术大会举办的频率和数量就一目了然了。

Go在国内真正上被全栈使用的就七牛一家，但国外除了docker，coreOS还有很多初创企业。
国内比较有影响力的就一个beego框架，你看看国外的有多少。

去github上搜一下active的Go的project数量，看看Go在国外是不是没人用？我反正在github的trending里面几乎每天都能看到Go的project。hacker news上面有关Go的“xxx writen in Go”的炒作文也不要太多。
这个 dariubs/GoBooks · GitHub 是有人整理的Go相关的书籍，看看是不是国外的书籍比国内的少？8月份K&R中的K也要推出属于Go的圣经了。

另外老有人喜欢说：Google喜欢关闭产品，这玩意儿迟早死掉。可惜golang是开源项目，关不掉的，CloudFlare那个crypto的patch(Gerrit Code Review)以后可能会进Go的标准库，Godep已经成为事实上的包管理标准，这些都是社区自己搞出来的，和google一毛线关系没有。另外就是最近google自己一些主力产品或者平台在优先支持语言上，Go总是和java，c/c++，python一起名列其中，grpc就是一个例子等等。所以，觉得Go只是google的一个玩具的人，你的观点能不能站得住脚，自己掂量吧。

我的个人观点是：
Go显示已经站住了脚跟(如果是2013年，我还是不敢说这种话的)，找到了属于自己的空间，但是比起那些主流的甚至nodeJS来说，还是使用的不够广泛。这个语言人为炒作也罢，一些人认为的google光环也罢，实际使用也罢，总之：
这个语言已经站住脚跟了，能用于并且已经用于生产环境了，接下来几年只会一直呈上升势头。


个人观点：

**1 一些真正使用go语言的公司：** 

这些公司在高速发展的同时，Golang也因此在国内逐渐传播开来。在云计算时代，从国内Go
语言发展和应用来看，七牛算是国内第一家选 Go
语言做服务端的公司。早在2011年，当Go语法还没完全稳定下来的情况下，七牛就已经选择将Go作为存储服务端的主题语言。关于这点，七牛CEO许式伟谈到：编程哲学的重塑是
Go 语言独树一帜的根本原因，其它语言仍难以摆脱 OOP
或函数式编程的烙印，只有 Go
完全放弃了这些，对编程范式重新思考，对热门的面向对象编程提供极度简约但却完备的支持。Go
是互联网时代的C语言，不仅会制霸云计算，10 年内将会制霸整个 IT 领域。

**2 在中国程序员眼中，谷歌出品必属精品** 
 确实，在互联网世界，在开源世界，Google为我们贡献了太多太多。

**3 创业公司假装高逼格，假装geek范儿** 
 The word geek is a slang term originally used to describe eccentric or
non-mainstream people; in current use, the word typically connotes an
expert or enthusiast or a person obsessed with a hobby or intellectual
pursuit, with a general pejorative meaning of a “peculiar person,
especially one who is perceived to be overly intellectual,
unfashionable, or socially awkward”.

**4 docker异常火爆，带动了对go语言的关注** 
 Docker是PaaS供应商dotCloud开源的一个基于LXC
的高级容器引擎，源代码托管在 GitHub 上, 基于Go语言开发并遵从Apache
2.0协议开源。

**5 go语言本身的一些特性** 
 部署简单 
 并发性好 
 性能好 
 。。。
