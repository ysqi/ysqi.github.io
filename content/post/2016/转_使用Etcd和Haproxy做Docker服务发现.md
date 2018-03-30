
---
date: 2016-12-31T11:34:39+08:00
title: "使用Etcd和Haproxy做Docker服务发现"
description: ""
disqus_identifier: 1485833679851249980
slug: "shi-yong--Etcd-he--Haproxy-zuo--Docker-fu-wu-fa-xian"
source: "https://segmentfault.com/a/1190000000730186"
tags: 
- architecture 
- golang 
- docker 
- haproxy 
- etcd 
topics:
- 编程语言与开发
---

使用 Etcd 和 Haproxy 做 Docker 服务发现
=======================================

标签（空格分隔）： Etcd Haproxy Docker 服务发现 architecture discovery
docker-gen golang service

------------------------------------------------------------------------

> 本文作者是 [jwilder](https://github.com/jwilder)，本文的原文是 [Docker
> Service Discovery Using Etcd and
> Haproxy](http://jasonwilder.com/blog/2014/07/15/docker-service-discovery/)

在前一篇文章中，我们展示了一种为 Docker
容器在同一台主机上创建一个[自动化 Nginx
反向代理](http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/)的方式。那个设置对于前端
web app
来说工作的很好，但是对于后端服务来说它不是一个好的点子，因为通常它们跨越多个主机。

这篇文章描述了一个为后端服务的 Docker 容器提供服务发现的解决方案。

我们将构建的架构体系是模仿
[SmartStack](http://nerds.airbnb.com/smartstack-service-discovery-cloud/)，但是使用
[etcd](http://coreos.com/using-coreos/etcd/) 代替
[Zookeeper](http://zookeeper.apache.org/)，和两个 docker 容器运行
[docker-gen](https://github.com/jwilder/docker-gen) 和
[haproxy](http://www.haproxy.org/) 代替
[nerve](https://github.com/airbnb/nerve) 和
[synapse](https://github.com/airbnb/synapse)。

它怎样工作的
------------

类似于
SmartStack，我们的组件服务作为一个注册（etcd），一个注册伙伴进程（docker-register），发现伙伴进程（docker-discover），一些后端服务（whoami）以及最后一个消费者（ubuntu/curl）。

注册和发现组件作为设备与应用程序容器工作，因此在后端或消费者容器的注册或发现代码不是被嵌入的。它们仅仅监听端口或连接其他本地端口。

服务注册 - Etcd
---------------

在任何事情被注册之前，我们需要一些地方跟踪注册条目（比如，服务的 IP
和端口）。我们使用 etcd，因为它由服务注册的简单程序模型和支持键的 TTLs
以及目录。

通常，你将运行 3到5个 etcd 节点，但是我们仅仅使用一个来保持事情简化。

没有理由为什么我们不能使用 [Consul](http://consul.io/)
或任何其他存储选项支持 TTL 过期。

开始 etcd：

    docker run -d --name etcd -p 4001:4001 -p 7001:7001 coreos/etcd

服务注册 - docker-register
--------------------------

注册服务容器被
[jwilder/docker-register](https://registry.hub.docker.com/u/jwilder/docker-register/)
容器处理。这个容器注册其他运行在同一台主机上的容器到 etcd
中。我们想注册的容器必须暴露一个端口。容器在不同的主机上运行相同的镜像是在
etcd
中被分组并将构成一个负载均衡集群。容器怎样分组是有点乱的，为这个演练我已经选择了容器镜像名字。在一个真实的部署中，你可能想通过环境变量，服务版本或其他的元数据分组。

（当前的实现仅仅支持每个容器一个端口并假设它当前是
TCP，没有理由为什么不能支持多个端口和类型以及不同的分组属性）

docker-register 使用
[docker-gen](https://github.com/jwilder/docker-gen)连同一个 [Python
脚本](https://github.com/jwilder/docker-register/blob/master/etcd.tmpl)作为一个模板。当运行的时候，动态的生成一个脚本，将在
`/backends` 目录注册每个容器的 IP 和端口。

docker-gen 关注监控 docker events和调用在一个间隔调用生成脚本来确保 TTLs
始终在最近的日期，如果 docker-register 停止了，注册过期。

为了启动 docker-register，我们需要传递主机的外部
IP，其他的主机能访问它的容器以及你的 etcd 主机的地址。 为了调用它的
API，docker-gen 要求访问 docker daemon，因此我们也绑定挂载 docker 的
unix socket 到容器中。

    HOST_IP=$(hostname --all-ip-addresses | awk '{print $1}')
    ETCD_HOST=w.x.y.z:4001
    docker run --name docker-register -d -e HOST_IP=$HOST_IP -e ETCD_HOST=$ETCD_HOST -v /var/run/docker.sock:/var/run/docker.sock -t jwilder/docker-register

服务发现 - docker-discover
--------------------------

服务发现被
[jwilder/docker-discover](https://registry.hub.docker.com/u/jwilder/docker-discover/)
容器处理。 docker-discover 周期性的投票 etcd
并通过监听每个服务类型来生成一个 haproxy 配置文件。

比如，容器运行
[jwilder/whoami](https://registry.hub.docker.com/u/jwilder/whoami/)
被注册在 `/backends/whoami/<id>`以及被暴露在主机上的端口是 8000。

其他的容器需要调用
[jwilder/whoami](https://registry.hub.docker.com/u/jwilder/whoami/)
服务，可以发送请求到 docker bridge IP:8000 或主机 IP:8000。

如果任何的后端服务宕了，haproxy
健康检查从池子中移除它并将在一台健康的主机上尝试请求。这确保后端服务可以随着需求被启动和停止，以及处理注册信息的不一致同时确保最小化的客户端影响。

最后，stats 可以通过在 docker-discover 容器上访问端口 1936 来查看。

运行 docker-discover：

    ETCD_HOST=w.x.y.z:4001
    docker run -d --net host --name docker-discover -e ETCD_HOST=$ETCD_HOST -p 127.0.0.1:1936:1936 -t jwilder/docker-discover

我们正在使用 `--net host` 以至于容器使用主机网络栈。当这个容器绑定 8000
端口，它实际是绑定在主机的网络上。这个简化了代理的设置。

AWS Demo
--------

我们将在 4 台 AWS 主机上运行整套服务：一台 etcd 主机, 一台 client 主机
和
两台后端主机。[后端服务](https://registry.hub.docker.com/u/jwilder/whoami/)
是一个简单的返回主机名的 golang HTTP 服务。

### Etcd 主机

首先，我们启动 etcd 注册：

    $ hostname --all-ip-addresses | awk '{print $1}'
    10.170.71.226

    $ docker run -d --name etcd -p 4001:4001 -p 7001:7001 coreos/etcd

我们的 etcd 地址是
`10.170.71.226`。我们将在其他主机上使用它。如果我们正在运行的是一个在线环境，我们可以分配一个
EIP 和 DNS 地址给它使得它更容易配置。

后端主机
--------

下一步，我们在每台主机上启动这个服务和
docker-register。该服务被配置成监听容器中的 8000 端口并且我们让 docker
把它发布在一台主机上的随机端口。

后端主机 1：

    $ docker run -d -p 8000:8000 --name whoami -t jwilder/whoami
    736ab83847bb12dddd8b09969433f3a02d64d5b0be48f7a5c59a594e3a6a3541
    $ docker run --name docker-register -d -e HOST_IP=$(hostname --all-ip-addresses | awk '{print $1}') -e ETCD_HOST=10.170.71.226:4001 -v /var/run/docker.sock:/var/run/docker.sock -t jwilder/docker-register
    77a49f732797333ca0c7695c6b590a64a7d75c14b5ffa0f89f8e0e21ae47ae3e

    $ docker ps
    CONTAINER ID        IMAGE                            COMMAND                CREATED             STATUS              PORTS                     NAMES
    736ab83847bb        jwilder/whoami:latest            /app/http              48 seconds ago      Up 47 seconds       0.0.0.0:49153->8000/tcp   whoami
    77a49f732797        jwilder/docker-register:latest   "/bin/sh -c 'docker-   28 minutes ago      Up 28 minutes                                 docker-register

后端主机 2：

    $ docker run -d -p 8000:8000 --name whoami -t jwilder/whoami
    4eb0498e52076275ee0702d80c0d8297813e89d492cdecbd6df9b263a3df1c28
    $ docker run --name docker-register -d -e HOST_IP=$(hostname --all-ip-addresses | awk '{print $1}') -e ETCD_HOST=10.170.71.226:4001 -v /var/run/docker.sock:/var/run/docker.sock -t jwilder/docker-register
    832e77c83591cb33bba53859153eb91d897f5a278a74d4ec1f66bc9b97deb221

    $ docker ps
    CONTAINER ID        IMAGE                            COMMAND                CREATED             STATUS              PORTS                     NAMES
    4eb0498e5207        jwilder/whoami:latest            /app/http              59 seconds ago      Up 58 seconds       0.0.0.0:49154->8000/tcp   whoami
    832e77c83591        jwilder/docker-register:latest   "/bin/sh -c 'docker-   34 minutes ago      Up 34 minutes                                 docker-register

### 客户端主机

在客户端主机，我们需要启动 docker-discover
和一个客户端服务。对于这个客户端容器，我使用 Ubuntu Trusty 并将做一些
`curl` 请求。

首先，启动 docker-discover：

    $ docker run -d --net host --name docker-discover -e ETCD_HOST=10.170.71.226:4001 -p 127.0.0.1:1936:1936 -t jwilder/docker-discover

然后，启动一个简单的客户端容器并传给它 `HOST_IP`。我们正在使用 eth0
地址，但也可以使用 docker0
IP。我们正以一个环境变量传给它因为[它是被配置的在两个部署之间变化的](http://12factor.net/config)。

    $ docker run -e HOST_IP=$(hostname --all-ip-addresses | awk '{print $1}') -i -t ubuntu:14.04 /bin/bash
    $ root@2af5f52de069:/# apt-get update && apt-get -y install curl

这时，构造一些请求给 whoami 服务端口 8000 来看他们的负载。

    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm 4eb0498e5207
    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm 736ab83847bb
    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm 4eb0498e5207
    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm 736ab83847bb

我们可以在后端启动一些实例：

    $ docker run -d -p :8000 --name whoami-2 -t jwilder/whoami
    $ docker run -d -p :8000 --name whoami-3 -t jwilder/whoami

    $ docker ps
    CONTAINER ID        IMAGE                            COMMAND                CREATED             STATUS              PORTS                     NAMES
    5d5c12c96192        jwilder/whoami:latest            /app/http              3 seconds ago       Up 1 seconds        0.0.0.0:49156->8000/tcp   whoami-2
    bb2a408b8ec5        jwilder/whoami:latest            /app/http              21 seconds ago      Up 20 seconds       0.0.0.0:49155->8000/tcp   whoami-3
    4eb0498e5207        jwilder/whoami:latest            /app/http              2 minutes ago       Up 2 minutes        0.0.0.0:49154->8000/tcp   whoami
    832e77c83591        jwilder/docker-register:latest   "/bin/sh -c 'docker-   36 minutes ago      Up 36 minutes                                 docker-register

然后再次在客户端主机上构造一些请求：

    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm 736ab83847bb
    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm 4eb0498e5207
    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm bb2a408b8ec5
    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm 5d5c12c96192
    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm 736ab83847bb

最后，我们关闭一些容器，路由将被更新。这个杀死在后端 2 的任何东西。

    $ docker kill 5d5c12c96192 bb2a408b8ec5 4eb0498e5207

    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm 736ab83847bb
    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm 67c3cccbb8ba
    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm 736ab83847bb
    $ root@2af5f52de069:/# curl $HOST_IP:8000
    I'm 67c3cccbb8ba

如果你想看 haproxy 是怎样负载流量的或监控错误，我们可以在 web
浏览器访问客户端主机的 1936 端口。

总结
----

虽然有不同的方式来实现[服务发现](http://jasonwilder.com/blog/2014/02/02/service-discovery-in-the-cloud/)，SmartStack
的伙伴注册行为和代理保持应用程序代码简单以及非常容易的融合进一个分布式环境，真的适合
Docker 容器。

同样地，Docker 的事件和容器 APIs 减轻了服务注册和使用注册服务发现（比如
etcd）的困难。

[docker-register](https://github.com/jwilder/docker-register) 和
[docker-discover](https://github.com/jwilder/docker-discover) 的代码在
github
上。虽然两个都是有用的，但是有很多地方需要提升。请随时提交或提出改进意见。

