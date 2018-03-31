
---
date: 2017-02-18T11:13:37+08:00
title: "CMDB发布平台:go发布管理"
description: ""
disqus_identifier: 1487387617412899783
slug: "CMDBfa-bu-ping-tai-:gofa-bu-guan-li"
source: "http://www.jianshu.com/p/4391eb7e894b"
tags: 
- golang 
categories:
- 编程语言与开发
---

CMDB发布平台是ezbuy的一个发布管理平台，包含了go的发布，windows
serices发布，iis发布，memcache管理，svn管理，资产信息管理操作。

随着公司的业务发展，公司go的服务有100多个单实例，如果多机部署计算，是成倍数增长的，而且每天更新发布的频率高，所以如果人为的去发布，会出现以下问题：

1.  开发人员每次update服务，都需要找运维人员发布
2.  每次发布，没有版本信息记载，只有相应的版本号，即没有历史数据
3.  重复工作量大，无技术可言，且容易人为发布错误
4.  没有消息通知
5.  开发人员各自在自己电脑上编译，然后提交到svn（环境不统一）

go现在的发布是每次jenkis自动编译通过后，自动上传到svn上，避免编译成mac
os版本的go上传到svn，然后通过CMDB平台发布，发布涉及手动和自动发布两个操作：

1.  手动：开发人员手动通过CMDB平台发布
2.  自动：编译好后，jenkis直接调用CMDB API自动发布到线上（持续CD）

综上问题，CMDB开发了go发布管理模块，模块里包含以下功能：

1.  部署
2.  发布
3.  更新go配置文件
4.  重启go服务
5.  版本回滚
6.  go服务状态
7.  go crontab发布更新
8.  go cron job定时任务列表

### [部署]：

线上运维人员可以通过运维平台直接部署一个新的go服务\

![](http://upload-images.jianshu.io/upload_images/258255-f9fed56a4f3330bc.png)\

go部署

### [发布]：

开发人员有权限直接发布服务到线上，无需运维人员干预，且每次发布要填入相应的tower发布url，否则不给予发布\

![](http://upload-images.jianshu.io/upload_images/258255-c4dde8405847a7d9.png?imageMogr2/auto-orient/strip%7CimageView2/2)\

go发布

\
不管服务成功与否，CMDB会调用钉钉api，将发布结果发送到钉钉消息群，同时CMDB也会保存一份日志到数据库。\

![](http://upload-images.jianshu.io/upload_images/258255-d9f4af13541f525d.png?imageMogr2/auto-orient/strip%7CimageView2/2)\

go发布消息

### [更新go配置文件]：

go配置文件我们会单独的将所有go实例的配置文件保存在一个svn库，这样的好处是，有版本控制，避免开发人员人为改动，且敏感信息只有相应权限的人才能查看，所以每次配置文件改表，直接相应有权限的人提交到svn，通过更新go配置文件下发的所有主机：\

![](http://upload-images.jianshu.io/upload_images/258255-88a20278c0f0a25c.png?imageMogr2/auto-orient/strip%7CimageView2/2)\

go下发配置文件

### [版本回滚]：

线上运维同学可以通过CMDB平台回滚到相应的版本

### [服务状态]：

开发人员可以实时查看go服务运行的状态

### [go crontab更新]：

开发人员可以直接发布go的服务

### [cron job列表]：

可以在CMDB平台查看所有的cron job和cron job上次执行的时间

最后，我们将会把go的配置文件统一线上线下环境一样，且不暴露敏感信息，敬请关注下一篇《如何利用consul-template保持线上线下配置文件一致》

