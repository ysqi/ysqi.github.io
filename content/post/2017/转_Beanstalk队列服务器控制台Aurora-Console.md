
---
date: 2017-05-24T09:17:27+08:00
title: "Beanstalk队列服务器控制台Aurora-Console"
description: ""
disqus_identifier: 1495588647816760092
slug: "Beanstalk-dui-lie-fu-wu-qi-kong-zhi-tai--Aurora-Console"
source: "https://segmentfault.com/a/1190000009357772"
tags: 
- golang 
- beanstalk 
- beanstalkd 
- queue 
- console 
categories:
- 编程语言与开发
---

aurora 是一个 Golang 编写的基于 Web 的 Beanstalk
消息队列服务器管理工具，单文件无需依赖其他组件，支持管理本地和远程多个队列服务器。

#### 项目地址

[github.com/Luxurioust/aurora](https://github.com/Luxurioust/aurora)

#### 特点

-   跨平台支持 macOS/Linux/Windows 32/64-bit

-   单文件简单易部署

-   不依赖其他组件

-   支持读取配置文件方式启动 + 登陆用户认证

-   定时刷新 Beanstalk 队列服务器状态

-   对每个 Tube 的 ready/delayed/buried 状态进行管理

-   支持批量清空 Tube 中的 Job

-   支持 Job 文本高亮显示

-   支持 Job 模糊搜索

-   自定义队列服务器状态监控项

#### 界面截图

-   Beanstalk 服务器列表

-   Tube 管理页面



