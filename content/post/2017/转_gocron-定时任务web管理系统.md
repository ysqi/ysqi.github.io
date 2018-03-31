
---
date: 2017-05-24T09:17:29+08:00
title: "gocron-定时任务web管理系统"
description: ""
disqus_identifier: 1495588649526781869
slug: "gocron---ding-shi-ren-wu-webguan-li-ji-tong"
source: "https://segmentfault.com/a/1190000009312368"
tags: 
- php 
- 定时任务 
- golang 
categories:
- 编程语言与开发
---

gocron - 定时任务web管理系统
============================

项目简介
========

使用Go语言开发的定时任务集中调度和管理系统, 用于替代Linux-crontab
[项目地址](https://github.com/ouqiang/gocron)
[查看文档](https://github.com/ouqiang/gocron/wiki)

功能特性
--------

-   支持任务CURD

-   crontab时间表达式，可精确到每秒

-   任务执行失败重试设置

-   任务超时设置

-   延时任务

-   任务执行方式

    -   调用本机系统命令

    -   通过SSH执行远程命令

    -   执行HTTP-GET请求

-   查看任务执行日志

-   任务执行结果通知, 支持邮件、Slack

### 截图

\

### 支持平台

> Windows、Linux、OSX

### 环境要求

> MySQL

安装
----

### 二进制安装

1.  解压压缩包

2.  `cd 解压目录`

3.  启动

    -   Windows: `gocron.exe web`

    -   Linux、OSX: `./gocron web`

4.  浏览器访问 <http://localhost:5920>

### 源码安装

1.  `go`语言版本1.7+

2.  `go get -d github.com/ouqiang/gocron`

3.  编译 `go build`

4.  启动、访问方式同上

### 启动可选参数

-   -p 端口, 指定端口, 默认5920

-   -e 指定运行环境, dev|test|prod, dev模式下可查看更多日志信息,
    默认prod

-   -h 查看帮助

安全
----

-   使用`https`访问保证数据传输安全,
    可在web服务器如nginx中配置https，通过反向代理，访问内部的gocron

-   网站访问设置IP白名单

-   SSH登录设置IP白名单



