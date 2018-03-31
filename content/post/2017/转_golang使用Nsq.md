
---
date: 2017-05-24T09:17:31+08:00
title: "golang使用Nsq"
description: ""
disqus_identifier: 1495588651040050362
slug: "golangshi-yong-Nsq"
source: "https://segmentfault.com/a/1190000009194607"
tags: 
- nsq 
- golang 
categories:
- 编程语言与开发
---

为什么要使用Nsq
===============

最近一直在寻找一个高性能，高可用的消息队列做内部服务之间的通讯。一开始想到用zeromq，但在查找资料的过程中，意外的发现了Nsq这个由golang开发的消息队列，毕竟是golang原汁原味的东西，功能齐全，关键是性能还不错。其中支持动态拓展，消除单点故障等特性， 
都可以很好的满足我的需求

下面上一张Nsq与其他mq的对比图，看上去的确强大。下面简单记录一下Nsq的使用方法\
\
图片来自golang2017开发者大会

Nsq服务端
=========

Nsq服务端简介
-------------

在使用Nsq服务之前，还是有必要了解一下Nsq的几个核心组件\
整个Nsq服务包含三个主要部分

### nsqlookupd

先看看官方的原话是怎么说：\
nsqlookupd是守护进程负责管理拓扑信息。客户端通过查询 nsqlookupd
来发现指定话题（topic）的生产者，并且 nsqd
节点广播话题（topic）和通道（channel）信息

简单的说nsqlookupd就是中心管理服务，它使用tcp(默认端口4160)管理nsqd服务，使用http(默认端口4161)管理nsqadmin服务。同时为客户端提供查询功能

总的来说，nsqlookupd具有以下功能或特性

-   唯一性，在一个Nsq服务中只有一个nsqlookupd服务。当然也可以在集群中部署多个nsqlookupd，但它们之间是没有关联的

-   去中心化，即使nsqlookupd崩溃，也会不影响正在运行的nsqd服务

-   充当nsqd和naqadmin信息交互的中间件

-   提供一个http查询服务，给客户端定时更新nsqd的地址目录 

### nsqadmin

官方原话：是一套 WEB UI，用来汇集集群的实时统计，并执行不同的管理任务

总的来说，nsqadmin具有以下功能或特性

-   提供一个对topic和channel统一管理的操作界面以及各种实时监控数据的展示，界面设计的很简洁，操作也很简单

-   展示所有message的数量，恩....装X利器

-   能够在后台创建topic和channel，这个应该不常用到

-   nsqadmin的所有功能都必须依赖于nsqlookupd，nsqadmin只是向nsqlookupd传递用户操作并展示来自nsqlookupd的数据

nsqadmin默认的访问地址是[http://127.0.0.1:4171/](http://127.0.0.1:4171/) 

### nsqd

官方原话：nsqd 是一个守护进程，负责接收，排队，投递消息给客户端

简单的说，真正干活的就是这个服务，它主要负责message的收发，队列的维护。nsqd会默认监听一个tcp端口(4150)和一个http端口(4151)以及一个可选的https端口

总的来说，nsqd 具有以下功能或特性

-   对订阅了同一个topic，同一个channel的消费者使用负载均衡策略（不是轮询）

-   只要channel存在，即使没有该channel的消费者，也会将生产者的message缓存到队列中（注意消息的过期处理）

-   保证队列中的message至少会被消费一次，即使nsqd退出，也会将队列中的消息暂存磁盘上(结束进程等意外情况除外)

-   限定内存占用，能够配置nsqd中每个channel队列在内存中缓存的message数量，一旦超出，message将被缓存到磁盘中

-   topic，channel一旦建立，将会一直存在，要及时在管理台或者用代码清除无效的topic和channel，避免资源的浪费

这是官方的图，第一个channel(meteics)因为有多个消费者，所以触发了负载均衡机制。后面两个channel由于没有消费者，所有的message均会被缓存在相应的队列里，直到消费者出现\

这里想到一个问题是，如果一个channel只有生产者不停的在投递message，会不会导致服务器资源被耗尽？也许nsqd内部做了相应处理，但还是要避免这种情况的出现

Nsq服务端与客户端的关系
-----------------------

了解nsqlookupd，nsqd与客户端中消费者和生产者的关系

#### 消费者

消费者有两种方式与nsqd建立连接

-   消费者直连nsqd，这是最简单的方式，缺点是nsqd服务无法实现动态伸缩了(当然，自己去实现一个也是可以的)
     

-   消费者通过http查询nsqlookupd获取该nsqlookupd上所有nsqd的连接地址，然后再分别和这些nsqd建立连接(官方推荐的做法)，但是客户端会不停的向nsqlookupd查询最新的nsqd地址目录(不喜欢用http轮询这种方式...)\
    还是看图更直接些 ，官方的消费者模型：

#### 生产者

生产者必须直连nsqd去投递message(网上说，可以连接到nsqlookupd，让nsqlookupd自动选择一个nsqd去完成投递，但是我用Producer的tcp是连不上nsqlookupd的，不知道http可不可以...)，

这里有一个问题就是如果生产者所连接的nsqd炸了，那么message就会投递失败，所以在客户端必须自己实现相应的备用方案

安装Nsq
-------

### 方法一

-   首先搭建golang开发环境，这里就不详细讲了

    -   注意一下，搭建golang环境时最好将bin目录添加到系统环境(path)里，省去了每次都要去bin目录里执行的麻烦

-   安装包管理器godep

<!-- -->

    go get github.com/tools/godep

执行完后检查godep是否已经安装在bin目录下，一般都会自动安装，如果没有，用go
install手动安装下

-   安装依赖包assert

<!-- -->

    go get github.com/bmizerany/assert

-   安装Nsq

<!-- -->

    godep get github.com/bitly/nsq/...

如果安装成功，bin目录里就会出现一大堆nsq\_...开头的可执行文件

-   PS：如果安装失败

    -   像我一样出现了一大堆"use of internal package not
        allowed"错误，找了半天，才在一个角落里发现了这句话\
        注意：NSQ 保持了 go get
        兼容，但是不推荐，因为之后不能保证仍然能稳定编译。

    这时采用方法二安装

### 方法二

-   直接去[https://github.com/nsqio/nsq/releases](https://github.com/nsqio/nsq/releases)下载编译好的执行文件，将里面的可执行文件复制到bin目录下就可以使用了

运行Nsq
-------

### 运行单机nsqd服务

nsqd是一个独立的服务，启动一个nsqd就可以完成message的收发，启动一个单机的nsqd，很简单

    nsqd

客户端可以使用http，也可以使用tcp，这里我使用是官方的go-nsq包做客户端，使用tcp进行message的收发

-   发送\

-   接收\

### 运行Nsq服务集群

-   首先启动nsqlookud

<!-- -->

    nsqlookupd

-   启动nsqd，并接入刚刚启动的nsqlookud。这里为了方便接下来的测试，启动了两个nsqd

<!-- -->

    nsqd --lookupd-tcp-address=127.0.0.1:4160

    nsqd --lookupd-tcp-address=127.0.0.1:4160 -tcp-address=0.0.0.0:4152 -http-address=0.0.0.0:4153

-   启动nqsadmin

<!-- -->

    nsqd --lookupd-http-address=127.0.0.1:4161

基于go-nsq的客户端实现
======================

几个值得注意的地方
------------------

-   Producer断线后不会重连，需要自己手动重连，Consumer断线后会自动重连

-   Consumer的重连时间配置项有两个功能(这个设计必须吐槽一下，分开配置更好一点)

    -   Consumer检测到与nsqd的连接断开后，每隔x秒向nsqd请求重连

    -   Consumer每隔x秒，向nsqlookud进行http轮询，用来更新自己的nsqd地址目录

    -   Consumer的重连时间默认是60s(...菜都凉了)，我改成了1s

-   Consumer可以同时接收不同nsqd
    node的同名topic数据，为了避免混淆，就必须在客户端进行处理

-   在AddConurrentHandlers和 AddHandler中设置的接口回调是在另外的goroutine中执行的

-   Producer不能发布(Publish)空message，否则会导致panic

go\_nsq-send.go
---------------

    //Nsq发送测试
    package main

    import (
     "bufio"
     "fmt"
     "github.com/nsqio/go-nsq"
     "os"
    )

    var producer *nsq.Producer

    // 主函数
    func main() {
     strIP1 := "127.0.0.1:4150"
     strIP2 := "127.0.0.1:4152"
     InitProducer(strIP1)

     running := true

     //读取控制台输入
     reader := bufio.NewReader(os.Stdin)
     for running {
      data, _, _ := reader.ReadLine()
      command := string(data)
      if command == "stop" {
       running = false
      }

      for err := Publish("test", command); err != nil; err = Publish("test", command) {
       //切换IP重连
       strIP1, strIP2 = strIP2, strIP1
       InitProducer(strIP1)
      }
     }
     //关闭
     producer.Stop()
    }

    // 初始化生产者
    func InitProducer(str string) {
     var err error
     fmt.Println("address: ", str)
     producer, err = nsq.NewProducer(str, nsq.NewConfig())
     if err != nil {
     panic(err)
     }
    }

    //发布消息
    func Publish(topic string, message string) error {
     var err error
     if producer != nil {
      if message == "" { //不能发布空串，否则会导致error
       return nil
      }
      err = producer.Publish(topic, []byte(message)) // 发布消息
      return err
     }
     return fmt.Errorf("producer is nil", err)
    }

go\_nsq-receive.go
------------------

    //Nsq接收测试
    package main

    import (
     "fmt"
     "time"

     "github.com/nsqio/go-nsq"
    )

    // 消费者
    type ConsumerT struct{}

    // 主函数
    func main() {
     InitConsumer("test", "test-channel", "127.0.0.1:4161")
     for {
      time.Sleep(time.Second * 10)
     }
    }

    //处理消息
    func (*ConsumerT) HandleMessage(msg *nsq.Message) error {
     fmt.Println("receive", msg.NSQDAddress, "message:", string(msg.Body))
     return nil
    }

    //初始化消费者
    func InitConsumer(topic string, channel string, address string) {
     cfg := nsq.NewConfig()
     cfg.LookupdPollInterval = time.Second          //设置重连时间
     c, err := nsq.NewConsumer(topic, channel, cfg) // 新建一个消费者
     if err != nil {
     panic(err)
     }
     c.SetLogger(nil, 0)        //屏蔽系统日志
     c.AddHandler(&ConsumerT{}) // 添加消费者接口

     //建立NSQLookupd连接
     if err := c.ConnectToNSQLookupd(address); err != nil {
     panic(err)
     }

     //建立多个nsqd连接
     // if err := c.ConnectToNSQDs([]string{"127.0.0.1:4150", "127.0.0.1:4152"}); err != nil {
     //  panic(err)
     // }

     // 建立一个nsqd连接
     // if err := c.ConnectToNSQD("127.0.0.1:4150"); err != nil {
     //  panic(err)
     // }
    }

