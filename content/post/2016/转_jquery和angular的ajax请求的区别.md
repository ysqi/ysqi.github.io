
---
date: 2016-12-31T11:35:08+08:00
title: "jquery和angular的ajax请求的区别"
description: ""
disqus_identifier: 1485833708289569013
slug: "jqueryhe-angularde-ajaxqing-qiu-de-ou-bie"
source: "https://segmentfault.com/a/1190000000396306"
tags: 
- revel 
- angular.js 
- jquery 
- golang 
categories:
- 编程语言与开发
---

最近用angular替换[我blog](http://115.29.47.52/post/index?stamp=1390187853)的部分页面。结果悲剧的发现，post请求到revel以后，revel的ParamsFilter解析不粗来参数。

看了下请求信息，发现jquery和angular的post请求是有些不同的。

jquery的content
type是application/x-www-form-urlencoded，会把post的参数拼接到url上，格式如foo=bar&baz=moe这样的。

而angular里，默认content type
是application/json，数据是以json格式发过去的。

但是在revel的params.go里面，没有对json格式的请求做参数处理。

用作者的原话说，这个json的处理没有什么用，而且在controller里用encoding/json处理也只是几行代码的事情。所以，就没有所以了。。。

关于这个问题的解决方法，有很多，可以从angular层面解决，把angular的post请求也按照jquery的方法做些改变，如下：

[](http://victorblog.com/2012/12/20/make-angularjs-http-service-behave-like-jquery-ajax/)<http://victorblog.com/2012/12/20/make-angularjs-http-service-behave-like-jquery-ajax/>

[](https://github.com/petersirka/total.js/issues/26)<https://github.com/petersirka/total.js/issues/26>

也可以从revel服务端解决。

[](https://github.com/robfig/revel/issues/97)<https://github.com/robfig/revel/issues/97>

本页的代码修改如下：

    func (c *Task) NewTask() revel.Result {

        decoder := json.NewDecoder(c.Request.Body)

        var content ToDoContent

        if err := decoder.Decode(&content); err != nil {

            print(">>>err:", err)

        } else {

            print(">>>>content:", content.Content)

        }

        json.Marshal(content)

        return c.RenderJson(content)

    }

虽然这样代码确实不多，不过瞬间感觉比rails弱爆了。。。

顺带的提一下，如果是jquery的请求，也需要稍微改动一下的，否则，revel一样的解析不粗来。

在jquery里要把post的jsonstringify一下。

具体参考这里[](https://github.com/robfig/revel/issues/126)<https://github.com/robfig/revel/issues/126>

    $.ajax({

      type:"POST", 

      url:"/Application/Index", 

      data:JSON.stringify({ name: "John", time: "2pm" }), 

      contentType:"application/json",

      dataType:"json"

    } )

