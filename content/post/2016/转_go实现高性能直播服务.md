
---
date: 2016-12-31T11:32:47+08:00
title: "go实现高性能直播服务"
description: ""
disqus_identifier: 1485833567483135222
slug: "goshi-xian-gao-xing-neng-zhi-bo-fu-wu"
source: "https://segmentfault.com/a/1190000007668492"
tags: 
- html5 
- javascript 
- shell 
- golang 
categories:
- 编程语言与开发
---

livego

go 实现直播服务

服务器部署

chmod 755 server.sh\
./server.sh &（依赖go环境，有些情况需要用vim 打开 set ff=unix 然后:wq）\
或者直接执行 ./LiveGoServer (不依赖go环境)

本地部署

直接执行 LiveGoServer.exe

use

采用vue+webpack实现ui\
所有在config里\
日志在logs里\
<http://localhost:8080/> (视频直播)\
<http://localhost:8080/camera>
(录视频)(由于chrome的安全限制，建议在firefox下打开录视频页面)

使用效果

交流使用

交流群：337937322

项目地址：[](https://github.com/qieangel2013/livego)[https://github.com/qieangel20...](https://github.com/qieangel2013/livego)

