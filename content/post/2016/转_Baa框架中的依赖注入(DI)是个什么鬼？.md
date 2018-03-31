
---
date: 2016-12-31T11:33:28+08:00
title: "Baa框架中的依赖注入(DI)是个什么鬼？"
description: ""
disqus_identifier: 1485833608345897527
slug: "Baakuang-jia-zhong-de-yi-lai-zhu-ru-(DI)shi-ge-shen-me-gui-？"
source: "https://segmentfault.com/a/1190000005702535"
tags: 
- di 
- baa 
- golang 
categories:
- 编程语言与开发
---

我最早接触的Go
WEB框架是[beego](http://beego.me/)，很强大的一个框架，也是很多人的首选，就是因为太（bu）强（gou）大（ling）了(huo)，后来尝试了[Macaron（martini）](https://go-macaron.com/)。Macaron的设计是众多框架的主流思想，路由、中间件、HTTP上下文，然后自己实现了一些常用的中间件（PS.
有一些中间件代码来自beego）。Macaron的思想中，可以通过m.Map()注入任意类型，然后在Context中通过反射获取这个类型，初试很爽，并为他的设计称赞。

在用PHP的时候有个框架 [Phalcon](https://phalconphp.com/)他的设计中核心是
[Dependency Injection/Service
Location](https://docs.phalconphp.com/en/latest/reference/di.html)，看起来很复杂，简单来说就是把类似log,db,cache,metadata等服务注册到DI中，使用的时候从DI取出来。Phalcon的使用姿势中就是先初始化一个APP，然后各种注册DI，然后RUN，伪代码如下：

    <?php

    $di = new \Phalcon\DI\FactoryDefault();
    $di->set('router', new MyRouter());
    $di->set('logger', function () {
        return new LoggerFile('../apps/logs/error.log');
    });
    $di->set('db', function () {
        return new PdoMysql(
            array(
                "host"     => "localhost",
                "username" => "root",
                "password" => "secret",
                "dbname"   => "blog"
            )
        );
    });
    $di->set('db2', ...);
    $di->set('mongo', new \MongoClient());

    // Create an application
    $application = new \Phalcon\Mvc\Application($di);

    // Handle the request
    echo $application->handle()->getContent();

上面的初始化，就是各种set，当然他支持set的类型比较丰富，还支持lazyload等，使用的方式也比较简单：

    <?php

    $di = new \Phalcon\DI\FactoryDefault();
    $db = $di->get('db');
    $mongo = $di->get('mongo');
    $mongo->selectCollection('xxx');

总结下来，就是
set/get，set就是设置一个服务（注入），get就是取出这个服务来使用，当然php不像Go是静态语言，他不需要做类型断言。

介绍完背景，其实也问题也基本明确了，在使用macaron的过程中，过分依赖反射，导致同一种类型只能注册一个，比如我要两个db，两个logger类型一致但做不同的服务用途，就比较难了。其实我最痛苦的是logger，macaron框架中使用了原声的log包，我无论怎么写都做不到日志统一，框架中输出的日志不依赖注入，就是原生的log。结合我对Phalcon的经验，就萌生了把Phalon
DI的思想移植过来的念头，再结合其他想法，就去做了Baa。DI是Baa中目前写得最简单的东西，但是却是最直接导致去造轮子的。

再来看Baa的DI思想特别简单，就是一个set一个get，使用姿势和Phalcon也一样，只不过目前没有提供那么多姿势的支持，比如懒加载（匿名函数）在调用时初始化，静态类就是set一个字符串使用的时候去new。虽然简单，但思想并无差别。我们可以做个示例：

    package main

    import (
        "github.com/go-baa/cache"
        _ "github.com/go-baa/cache/redis"
        "github.com/go-baa/render"
        "gopkg.in/baa.v1"
    )

    func main() {
        // new app
        app := baa.New()
        
        // register logger
        app.SetDI("logger", log.Logger)
        
        // register render
        b.SetDI("render", render.New(render.Options{
            Baa:        app,
            Root:       "templates/",
            Extensions: []string{".html", ".tmpl"},
            FuncMap:    template.Funcs(b),
        }))

        // register cache
        app.SetDI("cache", cache.New(cache.Options{
            Name:     "cache",
            Prefix:   "MyApp",
            Adapter:  "memory",
            Config:   map[string]string{},
        }))
        app.SetDI("cache2", cache.New(cache.Options{
            Name:     "cache2",
            Prefix:   "MyApp2",
            Adapter:  "redis",
            Config:   map[string]string{},
        }))

        // router
        app.Get("/", func(c *baa.Context) {
            ca := c.DI("cache").(cache.Cacher)
            ca.Set("test", "baa", 10)
            v := ca.Get("test").(string)
            c.String(200, v)
        })

        // run app
        app.Run(":1323")
    }

在app的初始化中通过set我们注入logger,render,cacher,甚至db的初始化等，在需要的地方比如Context中，context.DI()来获取，或者在其他地方可以
baa.Default().GetDI()来获取使用。再看我上面吐槽的log，在这里你只要`app.SetDI("logger", xxx)`即可以替换掉框架内置的logger，render也一样，只要注册一个实现了baa.Renderer接口的render就可以自定义模板引擎。

依赖注入(DI)不是什么玩意，虽然只有几行代码，但他是一种设计思想，一种理念，一种解决问题的姿势。

