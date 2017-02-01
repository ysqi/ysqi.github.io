
---
date: 2016-12-31T11:34:08+08:00
title: "shipyard中文版发布"
description: ""
disqus_identifier: 1485833648793472450
slug: "shipyardzhong-wen-ban-fa-bu"
source: "https://segmentfault.com/a/1190000003995350"
tags: 
- etcd 
- swarm 
- golang 
- docker 
- linux 
topics:
- 编程语言与开发
---

### shipyard中文版正式发布

> 一、 Docker Shipyard是什么
> ？shipyard是一个开源的docker管理平台，其特性主要包括：
>
> -   支持节点动态集群，可扩展节点的规模（swarm、etcd方案）
>
> -   支持镜像管理、容器管理、节点管理等功能
>
> -   可视化的容器管理和监控管理
>
> -   在线容器console终端
>
> 二、
> Shipyard中文版源代码托管在[github](https://github.com/cncodog/shipyard)，有兴趣可以访问git获取代码。下面是一些图片的预览：

        1.登陆界面

        2.容器列表

        3.容器信息创建

        4.容器信息

        5.终端连接

        6.镜像信息

        7.节点信息

        8.仓库信息

> 三、 如何使用

### 3.1如何安装shipyard中文版

      curl http://dockerclub.net/deploy |  bash -s 

### 3.2如何删除shipyard中文版

        curl http://dockerclub.net/deploy | ACTION =remove
     bash -s

### 3.2如何增加一个节点

        curl -sSL http://dockerclub.net/deploy | ACTION=node DISCOVERY=etcd://<你的首次安装主机IP> bash -s
     bash -s

