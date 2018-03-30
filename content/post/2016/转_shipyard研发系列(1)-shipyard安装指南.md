
---
date: 2016-12-31T11:34:08+08:00
title: "shipyard研发系列(1)-shipyard安装指南"
description: ""
disqus_identifier: 1485833648180721475
slug: "shipyardyan-fa-ji-lie-(1)-shipyardan-zhuang-zhi-na"
source: "https://segmentfault.com/a/1190000003995374"
tags: 
- golang 
- shipyard 
- docker 
topics:
- 编程语言与开发
---

### Docker之shipyard安装指南

> 一、前言：最近在研究shipyard的docker容器管理平台，在研究过程发现自己对docker基础理解还存在一定的欠缺，为了更好的理解docker，将我对shipyard项目的理解共享给docker爱好者，也系统的形成一份研发日志。[dockerclub的shipyard研发系列详细请访问](http://www.dockerclub.net)

> 二、shipyard项目介绍：shipyard是一个集成管理docker容器、镜像、Registries的系统，他最大亮点应该是支持多节点的集成管理，可以动态加载节点，可托管node下的容器。这里暂时不对shipyard的细节讲解，后续章节会讲他的架构。

> 三、shipyard安装使用介绍，shipyard项目的安装可以参考[官网](http://shipyard-project.com/docs/deploy/automated/)逐步操作，这里补充一些坑的解决过程。

### 3.1 首次部署脚本

    curl -sSL https://shipyard-project.com/deploy | bash -s

-   ACTION: 可以使用的指令 (deploy, upgrade, node, remove)

    1.  DISCOVERY: 集群系统采用Swarm进行采集和管理(在节点管理中可以使用‘node’)

    2.  IMAGE: 镜像，默认使用shipyard的镜像

    3.  PREFIX: 容器名字的前缀

    4.  SHIPYARD\_ARGS: 容器的常用参数

    5.  TLS\_CERT\_PATH: TLS证书路径

    6.  PORT: 主程序监听端口 (默认端口: 8080)

    7.  PROXY\_PORT: 代理端口 (默认: 2375)

### 3.2 脚本可选项

-   如果你要自定义部署，请参考以下规范

    1.  部署action：指令有效变量

    2.  deploy: 部署新的shipyard实例

    3.  upgrade:
        更新已存在的实例（注意：你要保持相同的系统环境、变量来部署同样的配置）

    4.  node: 使用Swarm增加一个新的node

    5.  remove: 删除已存在的shipyard实例（容器）

### 3.3 镜像使用

-   你可以采取规范的镜像来部署实例，比如以下的测试版本，你也已这样做

<!-- -->

     curl -sSL https://shipyard-project.com/deploy | IMAGE=shipyard/shipyard:test bash -s

### 3.4 前缀使用

-   你可以定义你想要的前缀，比如

<!-- -->

     curl -sSL https://shipyard-project.com/deploy | PREFIX=shipyard-test bash -s

### 3.5 参数使用

-   这里增加一些shipyard运行参数，你可以像这样进行调整：

<!-- -->

         curl -sSL https://shipyard-project.com/deploy | SHIPYARD_ARGS="--ldap-server=ldap.example.com --ldap-autocreate-users" bash -s

### 3.6TLS证书使用

-   启用TLS对组建进行部署，包括代理（proxy）、swarm集群系统、shipyard管理平台的配置，这是一个配置规范。证书必须采用以下命名规范：

    1.  ca.pem: 安全认证证书

    2.  server.pem: 服务器证书

    3.  server-key.pem: 服务器私有证书

    4.  cert.pem: 客户端证书

    5.  key.pem: 客户端证书的key

-   注意：证书将被放置在一个docker容器中，并在各个组成部分之间共享。如果需要调试，可以将此容器连接到调试容器。数据容器名称为前缀的证书。

<!-- -->

     docker run --rm \
        -v $(pwd)/certs:/certs \
        ehazlett/certm \
        -d /certs \
        bundle \
        generate \
        -o shipyard \
        --host proxy \
        --host 127.0.0.1

-   你也可以按如下指令来部署系统

<!-- -->

       curl -sSL https://shipyard-project.com/deploy | TLS_CERT_PATH=$(pwd)/certs bash -s

### 3.7增加一个部署节点

-   shipyard节点部署脚本将自动的安装key/value存储系统（etcd系统）。增加一个节点到swarm集群，你可以使用以下的节点部署脚本

<!-- -->

    curl -sSL https://shipyard-project.com/deploy | ACTION=node  DISCOVERY=etcd://10.0.1.10:4001 bash -s

-   注意：10.0.1.10这个ip地址你需要修改为你的首次初始化shipyard系统的主机地址

### 3.8删除shipyard系统

     curl -sSL https://shipyard-project.com/deploy | ACTION=remove bash -s

### 3.9 附件：

-   下面是笔者部署后的效果，如果你遇到问题，可以在dockerclub.net问答社区上给我留言。

    1.  启动界面\

    2.  容器详细情况\

    3.  镜像\

    4.  节点\



