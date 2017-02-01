
---
date: 2016-12-31T11:33:20+08:00
title: "open-falcon-hbs学习"
description: ""
disqus_identifier: 1485833600067764031
slug: "open-falcon-hbsxue-xi"
source: "https://segmentfault.com/a/1190000006069810"
tags: 
- go语言 
- golang 
topics:
- 编程语言与开发
---

open-falcon-hbs
===============

标签（空格分隔）： go falcon

------------------------------------------------------------------------

### 主要功能

-   处理agent心跳请求，填充host表

-   ip白名单下发所有agent

-   下发执行插件信息

-   下发监控端口、进程

-   缓存监控策略

### 模块结构

### 内存数据Map结构

-   HostMap:\
    `(hostname, hostId int)`

-   HostGroupsMap:\
    `(hostId, groupsId []int)`

-   GroupPlugins:\
    `(groupId, pluginsPath []string)`

-   GroupTemplates:\
    `(groupId, templatesID []int)`

-   TemplateCache:\
    `(templateId, Template)`

<!-- -->

    type Template struct {
        Id       int    `json:"id"`
        Name     string `json:"name"`
        ParentId int    `json:"parentId"`
        ActionId int    `json:"actionId"`
        Creator  string `json:"creator"`
    }

-   Strategies:\
    `(strategryID, Strategry)`

<!-- -->

    type Strategy struct {
        Id         int               `json:"id"`
        Metric     string            `json:"metric"`
        Tags       map[string]string `json:"tags"`
        Func       string            `json:"func"`       // e.g. max(#3) all(#3)
        Operator   string            `json:"operator"`   // e.g. < !=
        RightValue float64           `json:"rightValue"` // critical value
        MaxStep    int               `json:"maxStep"`
        Priority   int               `json:"priority"`
        Note       string            `json:"note"`
        Tpl        *Template         `json:"tpl"`
    }

-   HostTemplates:\
    `(hostID, templatesID []int)`

-   ExpressionCache:\
    `(expressionId, [] Expression)`

<!-- -->

    type Expression struct {
        Id         int               `json:"id"`
        Metric     string            `json:"metric"`
        Tags       map[string]string `json:"tags"`
        Func       string            `json:"func"`       // e.g. max(#3) all(#3)
        Operator   string            `json:"operator"`   // e.g. < !=
        RightValue float64           `json:"rightValue"` // critical value
        MaxStep    int               `json:"maxStep"`
        Priority   int               `json:"priority"`
        Note       string            `json:"note"`
        ActionId   int               `json:"actionId"`
    }

-   MonitoredHosts:\
    `(hostID, Host)`

<!-- -->

    type Host struct {
        Id   int
        Name string
    }

### DB和Cache

-   数据库操作

<!-- -->

    rows, err = DB.Query(sql)
    if err != nil {
        log.Println("ERROR:", err)
        return err
    }
    …
    defer DB.Close()
    for rows.Next(){
        …
        err = rows.Scan(&id, &hostname)
        if err != nil {
            log.Println("ERROR:", err)
            continue
        }
        …
    }

-   缓存策略\
    初始运行时从portalDB中读取数据结构数据存于内存中，然后每分钟执行一次portalDB查询（与初始运行操作一致）更新数据到内存中。

-   插件策略

1.  plugins update request
    包含hostname和checksum（checksum初始为空），HBS收到request后从portalDB中读取插件路径信息，排序后对路径取MD5形成checksum，与agent请求的checksum对比，相同则返回空不进行plugins更新，否则返回插件信息，agent收到后更新插件并定期执行插件。

### RPC实现方式

RPC服务可通过HTTP，TCP和JSON的方式建立Sever和Client；Hbs的RPC服务端通过JSON方式实现，可注册多个client，实现相关接口，供agent和judge调用。

