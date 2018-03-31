
---
date: 2016-12-31T11:33:40+08:00
title: "beego下根据部署环境加载相应配置文件"
description: ""
disqus_identifier: 1485833620328060013
slug: "beegoxia-gen-ju-bu-shu-huan-jing-jia-zai-xiang-ying-pei-zhi-wen-jian"
source: "https://segmentfault.com/a/1190000005063665"
tags: 
- beego 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

最近用beego开发的项目频繁的要部署到测试环境提测，然后部署到线上发布，由于两种环境下配置文件中某些配置参数不同，每次手动修改很是麻烦，故而想有没有办法能根据部署环境的不同加载相应环境的配置变量。\
幸而得同事告知，两种环境下都会注入ENV\_CLUSTER这个系统环境变量，且变量值不同。于是，在main函数beego.Run()执行前，我创建了一个预先执行的init文件，将自适应环境加载配置文件的代码逻辑放在了这里。\
具体代码如下：

        env := os.Getenv("ENV_CLUSTER")
        if env == "online" {
            beego.AppConfigPath = "./conf/online.conf"
        } else if env == "beta" {
            beego.AppConfigPath = "./conf/beta.conf"
        } else {
            beego.AppConfigPath = "./conf/dev.conf"
        }
        beego.ParseConfig()

