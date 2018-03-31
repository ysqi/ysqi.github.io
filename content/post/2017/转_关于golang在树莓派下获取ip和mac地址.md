
---
date: 2017-05-24T09:17:30+08:00
title: "关于golang在树莓派下获取ip和mac地址"
description: ""
disqus_identifier: 1495588650450120424
slug: "guan-yu-golangzai-shu-mei-pa-xia-huo-qu-iphe-macde-zhi"
source: "https://segmentfault.com/a/1190000009228676"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

前言
----

最近工作需要，需求为获取树莓派以太网ip\
地址和mac地址，看了下golang的文档，发现net.InterfaceByName可以完成这个目标。

实现
----

            //以太网网卡名称为eth0
            inter, err := net.InterfaceByName("eth0")
            if err != nil {
                    log.Fatalln(err)
            }
            //mac地址
            fmt.Println(inter.HardwareAddr.String())
            addrs, err := inter.Addrs()
            if err != nil {
                    log.Fatalln(err)
            }
            //ip地址一个ip4一个ip6
            for _, addr := range addrs {
                    fmt.Println(addr.String())
            }

运行结果：

后记
----

当然，树莓派3代自带无线网卡，名字换为wlan0就可以获取无线网卡ip。

