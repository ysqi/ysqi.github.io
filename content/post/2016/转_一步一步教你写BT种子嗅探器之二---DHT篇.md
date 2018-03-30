
---
date: 2016-12-31T11:33:14+08:00
title: "一步一步教你写BT种子嗅探器之二---DHT篇"
description: ""
disqus_identifier: 1485833594115726573
slug: "yi-bu-yi-bu-jiao-ni-xie-BTchong-zi-xiu-tan-qi-zhi-er----DHTpian"
source: "https://segmentfault.com/a/1190000006254137"
tags: 
- 搜索引擎 
- github 
- golang 
topics:
- 编程语言与开发
---

之前写了[原理篇](http://www.jianshu.com/p/5c8e1ef0e0c3)，在原理篇里简单的介绍了一下DHT，但是还不够详细。今天我们就专门详细的讲一下嗅探器的核心-DHT，这里默认原理篇你已经读了。

背景知识
--------

DHT全称 [Distributed Hash
Table](https://en.wikipedia.org/wiki/Distributed_hash_table)，中文翻译过来就是分布式哈希表。它是一种去中心化的分布式系统，特点主要有自动去中心化，强大的容错能力，支持扩展。另外它规定了自己的架构，包括keyspace和overlay
network（覆盖网络）两部分。但是他没有规定具体的算法细节，所以出现了很多不同的实现方式，比如Chord，Pastry，Kademlia等。BitTorrent中的DHT是基于Kademlia的一种变形，它的官方名称叫做
[Mainline DHT](https://en.wikipedia.org/wiki/Mainline_DHT)。

DHT人如其名，把它看成一个整体，从远处看它，它就是一张哈希表，只不过这张表是分布式的，存在于很多机器上。它同时支持set(key,
val)，get(key)操作。DHT可以用于很多方面，比如分布式文件系统，DNS，即时消息(IM)，以及我们最熟悉的点对点文件共享（比如BT协议）等。

下面我们提到的DHT默认都是Mainline
DHT，例子都是用伪代码来表示。**读下面段落的时候要时刻记着，DHT是一个哈希表**。

Mainline DHT
------------

Mainline DHT遵循DHT的架构，下面我们分别从Keyspace和Overlay
network两方面具体说明。

### Keyspace

keyspace主要是关于key的一些规定。

Mainline
dht里边的key长度为160bit，注意是bit，不是byte。在常见的编译型编程语言中，最长的整型也才是64bit，所以用整型是表示不了key的，我们得想其他的方式。我们可以用数组方式表示它，数组类型你可以选用长度不同的整型，比如int8，int16，int32等。这里为了下边方便计算，我们采用长度为20的byte数组来表示。

在mainline
dht中，key之间唯一的一种计算是xor，即异或（还记得异或的知识吧？）。我们的key是用长度为20的byte数组来表示，因此我们应该从前往后依次计算两个key的相对应的byte的异或值，最终结果得到的是另外一个长度为20的byte数组。算法如下：

    ​for i = 0; i < 20; i++ {
    ​    result[i] = key1[i] ^ key2[i];
    ​}

读到这里，你是不是要问xor有啥用？还记得[原理篇](http://www.jianshu.com/p/5c8e1ef0e0c3)中DHT的工作方式吗？

xor是为了找到好友表中离key最近的k个节点，什么样的节点最近？就是好友中每个节点和key相异或，得到的结果越小就越近。这里又衍生另外一个问题，byte数组之间怎么比较大小？很简单，从前往后，依次比较每一个byte的大小即可。

在Mainline
DHT中，我们用160bit的key来代表每个节点和每个资源的ID，我们查找节点或者查找资源的时候实际上就是查找他们的ID。回想一下，这是不是很哈希表?
:)

另外聪明的你可能又该问了，我们怎么样知道每个节点或者每个资源的ID是多少？在Mainline
DHT中，节点的ID一般是随机生成的，而资源的ID是用sha1算法加密资源的内容后得到的。

OK，关于key就这么多，代码实现你可以查考[这里](https://github.com/shiyanhui/dht/blob/master/bitmap.go)。

### Overlay network

Overlay
network主要是关于DHT内部节点是怎么存储数据的，不同节点之间又是怎样通信的。

首先我们回顾一下原理篇中DHT的工作方式:

> DHT
> 由很多节点组成，每个节点保存一张表，表里边记录着自己的好友节点。当你向一个节点A查询另外一个节点B的信息的时候，A就会查询自己的好友表，如果里边包含B，那么A就返回B的信息，否则A就返回距离B距离最近的k个节点。然后你再向这k个节点再次查询B的信息，这样循环一直到查询到B的信息，查询到B的信息后你应该向之前所有查询过的节点发个通知，告诉他们，你有B的信息。

整个DHT是一个哈希表，它把自己的数据化整为零分散在不同的节点里。OK，现在我们看下，一个节点内部是用什么样的数据结构存储数据的。

#### 节点内部数据存储 - Routing Table

用什么样的数据结构得看支持什么样的操作，还得看各种操作的频繁程度。从上面工作方式我们知道，操作主要有两个：

-   在我（注意：“我”是一个节点）的好友节点中查询离一个key最近的k个节点（在Mainline
    DHT中，k=8），程度为频繁

-   把一个节点保存起来，也就是插入操作，程度为频繁

首先看到“最近”、“k”，我们会联想到top
k问题。一个很straightforward的做法是，用一个数组保存节点。这样的话，我们看下算法复杂度。top
k问题用堆解决，查询复杂度为O(k +
(n-k)\*log(k))，当k=8时，接近于O(n)；插入操作为O(1)。注：n为一个节点的好友节点总数。

当n很大的时候，操作时间可能会很长。那么有没有O(log(n))的算法呢？

联想到上面堆的算法，你可能说，我们可以维护一个堆啊，插入和查询的时候都是O(log(n))。这种做法堆是根据堆中元素与某一个固定不变的key的距离来维护的，但是通常情况下，我们查询的key都是变化的，因此这种做法不可行。

那还有其他O(log(n))的算法吗？

经验告诉我们，很多O(log(n))的问题都和二叉树相关，比如各种平衡二叉树，我们能不能用二叉树来解决呢？联想到每个ID都是一个160bit的值，而且我们知道key之间的距离是通过异或来计算的，并且两个key的异或结果大小和他们的共同前缀无关，我们应该想到用Trie树（或者叫前缀树）来解决。事实上，Mainline
DHT协议中用的就是Trie树，但是与Trie树又稍微有所不同。在Trie树里边，插入一个key时，我们要比对key的每一个char和Trie里边路径，当不一致时，会立刻分裂成一个子树。但是在这里，当不一致时，不会立刻分裂，而是有一个长度为k的buffer（在Mainline
DHT中叫bucket）。分两种情况讨论：

-   如果bucket长度小于k，那么直接插入bucket就行了。

-   如果bucket长度大于或等于k，又要分两种情况讨论：

    -   第一种情况是当前的路径是该节点ID（注意不是要插入的key，是“我”自己的ID）的前缀，那么就分裂，左右子树的key分别是0和1，并且把当前bucket中的节点根据他们的当前char值分到相应的子树的bucket里边。

    -   第二种情况是当前路径不是该节点ID的前缀，这种情况下，直接把这个key丢掉。

如果还没有理解，你可以参照[Kademlia](http://www.ic.unicamp.br/~bit/ensino/mo809_1s13/papers/P2P/Kademlia-%20A%20Peer-to-Peer%20Information%20System%20Based%20on%20the%20XOR%20Metric%20.pdf)这篇论文上面的图。

插入的时候，复杂度为O(log(n))。查询离key最近的k个节点时，我们可以先找到当前key对应的bucket，如果bucket里边不够k个，那么我们再查找该节点前驱和后继，最后根据他们与key的距离拍一下序即可，平均复杂度也为O(log(n))。这样插入和查询都是O(log(n))。

代码实现你可以查考[这里](https://github.com/shiyanhui/dht/blob/master/routingtable.go)。

#### 节点之间的通信 - KRPC

KRPC比较简单，它是一个简单的rpc结构，其是通过UDP传送消息的，报文是由bencode编码的字典。它包含3种消息类型，request、response和error。请求又分为四种：ping，find\_node,
get\_peers, announce\_peer。

-   ping 用来侦探对方是否在线

-   find\_node
    用来查找某一个节点ID为Key的具体信息，信息里包括ip，port，ID

-   get\_peers
    用来查找某一个资源ID为Key的具体信息，信息里包含可提供下载该资源的ip:port列表

-   announce\_peer
    用来告诉别人自己可提供某一个资源的下载，让别人把这个消息保存起来。还记得Angelababy那个例子吗？在我得到她的微信号后，我会通知所有我之前问过的人，他们就会把我有Angelababy微信号这个信息保存起来，以后如果有人再问他们有没有Angelababy微信号的话，他们就会告诉那个人我有。**BT种子嗅探器就是根据这个来得到消息的，不过得到消息后我们还需要进一步下载**。

跳出节点，整体看DHT这个哈希表，find\_node和get\_peers就是我们之前说的get(key)，announce\_peer就是set(ke,
val)。

剩下的就是具体的消息格式，你可以在[官方文档](http://www.bittorrent.org/beps/bep_0005.html)上看到，这里就不搬砖了。

实现KRPC时，需要注意的有以下几点：

-   每次收到请求或者回复你都需要根据情况更新你的Routing
    Table，或保存或丢掉。

-   你需要实现transaction，transaction里边要包含你的请求信息以及被请求的ip及端口，只有这样当你收到回复消息时，你才能根据消息的transaction
    id做出正确的处理。Mainline
    DHT对于如何实现transaction没有做具体规定。

-   一开始你是不在DHT网络中的，你需要别人把你介绍进去，任何一个在DHT中的人都可以。一般我们可以向
    **router.bittorrent.com:6881**、 **dht.transmissionbt.com:6881**
    等发送find\_node请求，然后我们的DHT就可以开始工作了。

KRPC的实现你可以参考[这里](https://github.com/shiyanhui/dht/blob/master/krpc.go)。

总结
----

DHT整体就是一张哈希表，首先我们本身是里边的一个节点，我们向别人发送krpc
find\_node或get\_peers消息，就是在对这个哈希表执行get(key)操作。向别人发送announce\_peer消息，就是在对这个哈希表执行set(key,
val)操作。

最后
----

<https://github.com/shiyanhui/dht> 完整代码在这里，喜欢这篇文章的话就到github上给个Star呗
:)

[http://bthub.io](http://bthub.io/) 是基于上面这个嗅探器写的一个BT种子搜索引擎。

有任何问题可以在这里提问：<https://github.com/shiyanhui/dht/issues>

OK，今天就说到这里，关于怎么样下载，我们下篇再说。

你可以关注我的公众号，及时获得下一篇推送。



