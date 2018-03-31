
---
date: 2016-12-31T11:34:21+08:00
title: "go语言goget导入官方依赖的解决方法"
description: ""
disqus_identifier: 1485833661964400062
slug: "goyu-yan-go-get-dao-ru-guan-fang-yi-lai-de-jie-jue-fang-fa"
source: "https://segmentfault.com/a/1190000003509198"
tags: 
- 科学上网 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

由于众所周知的原因，使用go语言的时候会发生这样，那样的问题。比如使用go
get 导入官方依赖的时候回报错。\
再次感谢伟大的xxwall.\
那么，如何才能绕过这道门槛呢？\
解决方案

    1： 使用 http_proxy环境变量
        再执行 go get 之前，先使用 set 命令设置一下 环境变量，使用http_proxy 这个环境变量制定一个科学上网的proxy,再go get 就没问题了。
        set http_proxy=127.0.0.1:8787
        go get 
        ...
    2: 从github上面导入，再手动copy到%GOPATH%\src\golang.org\x\xxx 下面，这样就会从本地导入包了。
    比如：
        go get github.com/golang/net
        再拷贝到上面说的目录下即可。
        

