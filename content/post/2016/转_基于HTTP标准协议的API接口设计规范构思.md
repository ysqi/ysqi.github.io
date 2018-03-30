
---
date: 2016-12-31T11:32:42+08:00
title: "基于HTTP标准协议的API接口设计规范构思"
description: ""
disqus_identifier: 1485833562468197683
slug: "ji-yu-HTTPbiao-zhun-xie-yi-de-APIjie-kou-she-ji-gui-fan-gou-sai"
source: "https://segmentfault.com/a/1190000007855820"
tags: 
- node.js 
- python 
- javascript 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

开发规范
--------

1.  版本控制git

2.  开发流程git flow

接口
----

  请求方式   url                动作              中文说明
  ---------- ------------------ ----------------- ----------
  GET        `/resources/`      list              列表
  POST       `/resources/`      create            创建
  GET        `/resources/:id`   retrieve          详细
  PUT        `/resources/:id`   update            更新
  PATCH      `/resources/:id`   partial\_update   部分更新
  DELETE     `/resources/:id`   destroy           删除

数据
----

1.  请求支持form-date,json,x-www-form-urlencode

2.  返回格式统一为json

3.  一个请求对应一个serializer

错误
----

1.  错误信息包含在返回内容里

2.  不同的错误对应不同的错误信息代码

3.  http错误码按照标准用法使用

认证
----

1.  jwt

2.  token

3.  oauth2

权限
----

1.  以中间件形式作为权限鉴别插件，根据http请求格式直接判断权限

2.  用户登录成功时，将用户信息与权限信息缓存保证效率

日志
----

1.  日志以中间件形式提供

2.  根据业务需求氛围入库日志与普通日志

文档(待完善)
------------

根据上面的接口格式写文档

    {
      "resources": {
        "list": {
          "params": {},
          "response": {}
        },
        "create": {
          "request": {},
          "response": {}
        },
        "retrieve": {
          "response": {}
        },
        "update": {
          "request": {},
          "response": {}
        },
        "partial_update": {
          "request": {},
          "response": {}
        },
        "destroy": {}
      }
    }

测试
----

业务所需接口测试覆盖率100%

部署
----

-   docker

-   docker-compose

-   docker-machine

-   docker-swarm

服务器资源监控
--------------

待完善

