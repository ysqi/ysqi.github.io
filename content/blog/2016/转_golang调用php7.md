
---
date: 2016-12-31T11:32:47+08:00
title: "golang调用php7"
description: ""
disqus_identifier: 1485833567994671399
slug: "golang-diao-yong--php7"
source: "https://segmentfault.com/a/1190000007619087"
tags: 
- golang 
- php 
- php7 
topics:
- 编程语言与开发
---

使用
[`https://github.com/taowen/go-php7`](https://github.com/taowen/go-php7)\
基于 <https://github.com/deuill/go-php>
修改而来，fork缘由（[https://github.com/deuill/go-...](https://github.com/deuill/go-php/issues/32)）

执行php文件
===========

    func Test_exec(t *testing.T) {
        engine.Initialize()
        ctx := &engine.Context{
            Output: os.Stdout,
        }
        err := engine.RequestStartup(ctx)
        if err != nil {
            fmt.Println(err)
        }
        defer engine.RequestShutdown(ctx)
        err = ctx.Exec("/tmp/index.php")
        if err != nil {
            fmt.Println(err)
        }
    }

其中 `/tmp/index.php` 的内容为

    <?php
    echo("hello\n");

Eval，返回值
============

    func Test_eval(t *testing.T) {
        engine.Initialize()
        ctx := &engine.Context{}
        err := engine.RequestStartup(ctx)
        if err != nil {
            fmt.Println(err)
        }
        defer engine.RequestShutdown(ctx)
        val, err := ctx.Eval("return 'hello';")
        if err != nil {
            fmt.Println(err)
        }
        defer engine.DestroyValue(val)
        if engine.ToString(val) != "hello" {
            t.FailNow()
        }
    }

返回的value的生命周期所有权是golang程序，所以我们要负责DestroyValue

设置全局变量来传参
==================

    func Test_argument(t *testing.T) {
        engine.Initialize()
        ctx := &engine.Context{}
        err := engine.RequestStartup(ctx)
        if err != nil {
            fmt.Println(err)
        }
        defer engine.RequestShutdown(ctx)
        err = ctx.Bind("greeting", "hello")
        if err != nil {
            fmt.Println(err)
        }
        val, err := ctx.Eval("return $greeting;")
        if err != nil {
            fmt.Println(err)
        }
        defer engine.DestroyValue(val)
        if engine.ToString(val) != "hello" {
            t.FailNow()
        }
    }

传递进去的参数的生命周期是php控制的，在request
shutdown的时候内存会被释放。

PHP 回调 Golang
===============

    type greetingProvider struct {
        greeting string
    }

    func (provider *greetingProvider) GetGreeting() string {
        return provider.greeting
    }

    func newGreetingProvider(args []interface{}) interface{} {
        return &greetingProvider{
            greeting: args[0].(string),
        }
    }

    func Test_callback(t *testing.T) {
        engine.Initialize()
        ctx := &engine.Context{}
        err := engine.RequestStartup(ctx)
        if err != nil {
            fmt.Println(err)
        }
        defer engine.RequestShutdown(ctx)
        err = engine.Define("GreetingProvider", newGreetingProvider)
        if err != nil {
            fmt.Println(err)
        }
        val, err := ctx.Eval(`
        $greetingProvider = new GreetingProvider('hello');
        return $greetingProvider->GetGreeting();`)
        if err != nil {
            fmt.Println(err)
        }
        defer engine.DestroyValue(val)
        if engine.ToString(val) != "hello" {
            t.FailNow()
        }
    }

PHP 错误日志
============

    func Test_log(t *testing.T) {
        engine.PHP_INI_PATH_OVERRIDE = "/tmp/php.ini"
        engine.Initialize()
        ctx := &engine.Context{
            Log: os.Stderr,
        }
        err := engine.RequestStartup(ctx)
        if err != nil {
            fmt.Println(err)
        }
        defer engine.RequestShutdown(ctx)
        _, err = ctx.Eval("error_log('hello', 4); trigger_error('sent from golang', E_USER_ERROR);")
        if err != nil {
            fmt.Println(err)
        }
    }

其中 `/tmp/php.ini` 的内容为

    error_reporting = E_ALL
    error_log = "/tmp/php-error.log"

错误会被输出到
/tmp/php-error.log。直接调用error\_log会同时再输出一份到stderr

HTTP 输入输出
=============

    func Test_http(t *testing.T) {
        engine.Initialize()
        recorder := httptest.NewRecorder()
        ctx := &engine.Context{
            Request: httptest.NewRequest("GET", "/hello", nil),
            ResponseWriter: recorder,
        }
        err := engine.RequestStartup(ctx)
        if err != nil {
            fmt.Println(err)
        }
        defer engine.RequestShutdown(ctx)
        _, err = ctx.Eval("echo($_SERVER['REQUEST_URI']);")
        if err != nil {
            fmt.Println(err)
        }
        body, err := ioutil.ReadAll(recorder.Result().Body)
        if err != nil {
            fmt.Println(err)
        }
        if string(body) != "/hello" {
            t.FailNow()
        }
    }

所有的PHP超级全局变量都会被初始化为传递进去的Request的值，包括

    $_SERVER
    $_GET
    $_POST
    $_FILE
    $_COOKIE
    $_ENV

echo的内容，http code和http header会被写回到传入的ResponseWriter

fastcgi\_finish\_request
========================

PHP-FPM
很常用的一个功能是`fastcgi_finish_request`，用于在php里做一些异步完成的事情。这个特殊的全局函数必须支持

    func Test_fastcgi_finish_reqeust(t *testing.T) {
        engine.Initialize()
        buffer := &bytes.Buffer{}
        ctx := &engine.Context{
            Output: buffer,
        }
        err := engine.RequestStartup(ctx)
        if err != nil {
            fmt.Println(err)
        }
        defer engine.RequestShutdown(ctx)
        ctx.Eval("ob_start(); echo ('hello');")
        if buffer.String() != "" {
            t.FailNow()
        }
        ctx.Eval("fastcgi_finish_request();")
        if buffer.String() != "hello" {
            t.FailNow()
        }
    }

实际的作用就是把output提前输出到 ResposneWriter
里去，让调用方知道结果。对于当前进程的执行其实是没有影响的，只是影响了output。

