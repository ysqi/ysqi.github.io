
---
date: 2016-12-31T11:34:04+08:00
title: "PHP编程中的并发"
description: ""
disqus_identifier: 1485833644832294466
slug: "PHPbian-cheng-zhong-de-bing-fa"
source: "https://segmentfault.com/a/1190000004069411"
tags: 
- golang 
- 线程 
- 进程 
- 并发 
- php 
topics:
- 编程语言与开发
---

PHP编程中的并发
===============

> 周末去北京面了两个公司，认识了几位技术牛人，面试中聊了很多，感觉收获颇丰。认识到了自己的不足之处，也坚定了自己对计算机学习的信心。本文是对其中一道面试题的总结。

面试中有一个问题没有很好的回答出来，题目为：并发3个http请求，只要其中一个请求有结果，就返回，并中断其他两个。

当时考虑的内容有些偏离题目原意， 一直在考虑如何中断http请求，大概是在
`client->recv()` 之前去判断结果是否已经产生，所以回答的是用 socket
去发送一个 http 请求，把 socket 加入 libevent
循环监听，在callback中判断是否已经得到结果，如果已经得到结果，就直接
return。

后来自己越说越觉得不对，既然已经recv到结果，就不能算是中断http请求。何况自己从来没用过libevent。后来说了还说了两种实现，一个是用
`curl_multi_init`, 另一个是用golang实现并发。\
golang的版本当时忘了`close`的用法，结果并不太符合题意。

这题没答上来，考官也没为难我。但是心里一直在考虑，直到面试完走到楼下有点明白什么意思了，可能考的是并发，进程线程的应用。所以总结了这篇文章，来讲讲PHP中的并发。\
本文大约总结了PHP编程中的五种并发方式，最后的Golang的实现纯属无聊，可以无视。如果有空，会再补充一个libevent的版本。

curl\_multi\_init
-----------------

文档中说的是
`Allows the processing of multiple cURL handles asynchronously.`
确实是异步。这里需要理解的是`select`这个方法，文档中是这么解释的`Blocks until there is activity on any of the curl_multi connections.`。了解一下常见的异步模型就应该能理解，select,
epoll，都很有名，这里引用[一篇非常好的文章](http://segmentfault.com/a/1190000003063859)，有兴趣看下解释吧。

    <?php
    // build the individual requests as above, but do not execute them
    $ch_1 = curl_init('http://www.baidu.com/');
    $ch_2 = curl_init('http://www.baidu.com/');
    curl_setopt($ch_1, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch_2, CURLOPT_RETURNTRANSFER, true);

    // build the multi-curl handle, adding both $ch
    $mh = curl_multi_init();
    curl_multi_add_handle($mh, $ch_1);
    curl_multi_add_handle($mh, $ch_2);

    // execute all queries simultaneously, and continue when all are complete
    $running = null;
    do {
       curl_multi_exec($mh, $running);
       $ch = curl_multi_select($mh);
       if($ch !== 0){
           $info = curl_multi_info_read($mh);
           if($info){
               var_dump($info);
               $response_1 = curl_multi_getcontent($info['handle']);
               echo "$response_1 \n";
               break;
           }
       }
    } while ($running > 0);

    //close the handles
    curl_multi_remove_handle($mh, $ch_1);
    curl_multi_remove_handle($mh, $ch_2);
    curl_multi_close($mh);

这里我设置的是，select得到结果，就退出循环，并且删除 `curl resource`,
从而达到取消http请求的目的。

swoole\_client
--------------

`swoole_client`提供了异步模式，我竟然把这个忘了。这里的sleep方法需要swoole版本大于等于1.7.21,
我还没升到这个版本，所以直接exit也可以。

    <?php
    $client = new swoole_client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);
    //设置事件回调函数
    $client->on("connect", function($cli) {
        $req = "GET / HTTP/1.1\r\n
        Host: www.baidu.com\r\n
        Connection: keep-alive\r\n
        Cache-Control: no-cache\r\n
        Pragma: no-cache\r\n\r\n";

        for ($i=0; $i < 3; $i++) {
            $cli->send($req);
        }
    });
    $client->on("receive", function($cli, $data){
        echo "Received: ".$data."\n";
        exit(0);
        $cli->sleep(); // swoole >= 1.7.21
    });
    $client->on("error", function($cli){
        echo "Connect failed\n";
    });
    $client->on("close", function($cli){
        echo "Connection close\n";
    });
    //发起网络连接
    $client->connect('183.207.95.145', 80, 1);

process
-------

哎，竟然忘了 swoole\_process, 这里就不用 pcntl
模块了。但是写完发现，这其实也不算是中断请求，而是哪个先到读哪个，忽视后面的返回值。

    <?php

    $workers = [];
    $worker_num = 3;//创建的进程数
    $finished = false;
    $lock = new swoole_lock(SWOOLE_MUTEX);

    for($i=0;$i<$worker_num ; $i++){
        $process = new swoole_process('process');
        //$process->useQueue();
        $pid = $process->start();
        $workers[$pid] = $process;
    }

    foreach($workers as $pid => $process){
        //子进程也会包含此事件
        swoole_event_add($process->pipe, function ($pipe) use($process, $lock, &$finished) {
            $lock->lock();
            if(!$finished){
                $finished = true;
                $data = $process->read();
                echo "RECV: " . $data.PHP_EOL;
            }
            $lock->unlock();
        });
    }

    function process(swoole_process $process){
        $response = 'http response';
        $process->write($response);
        echo $process->pid,"\t",$process->callback .PHP_EOL;
    }

    for($i = 0; $i < $worker_num; $i++) {
        $ret = swoole_process::wait();
        $pid = $ret['pid'];
        echo "Worker Exit, PID=".$pid.PHP_EOL;
    }

pthreads
--------

编译pthreads模块时，提示php编译时必须打开ZTS, 所以貌似必须 thread safe
版本才能使用. wamp中多php正好是TS的，直接下了个dll,
文档中的说明复制到对应目录，就在win下测试了。 还没完全理解，查到文章说
php 的 pthreads 和 POSIX
pthreads是完全不一样的。代码有些烂，还需要多看看文档，体会一下。

    <?php
    class Foo extends Stackable {
        public $url;
        public $response = null;
        public function __construct(){
            $this->url = 'http://www.baidu.com';
        }
        public function run(){}
    }

    class Process extends Worker {
        private $text = "";
        public function __construct($text,$object){
            $this->text = $text;
            $this->object = $object;
        }
        public function run(){
            while (is_null($this->object->response)){
                print " Thread {$this->text} is running\n";
                $this->object->response = 'http response';
                sleep(1);
            }
        }
    }

    $foo = new Foo();

    $a = new Process("A",$foo);
    $a->start();

    $b = new Process("B",$foo);
    $b->start();

    echo $foo->response;

yield
-----

yield生成的generator,可以中断函数，并用send向 generator 发送消息。\
稍后补充协程的版本。还在学习中。

Golang
------

用Go实现比较简单, 回家后查了查 close，处理一下 panic就ok了。代码如下：

    package main

    import (
        "fmt"
    )

    func main() {
        var result chan string = make(chan string, 1)
        for index := 0;  index< 3; index++ {
            go doRequest(result)
        }

        res, ok := <-result
        if ok {
            fmt.Println("received ", res)
        }

    }

    func doRequest(result chan string)  {
        response := "http response"
        defer func() {
            if x := recover(); x != nil {
                fmt.Println("Unable to send: %v", x)
            }
        }()
        result <- response
        close(result)
    }

上面的几个方法，除了 `curl_multi_*`
貌似符合题意外(不确定，要看下源码)，其他的方法都没有中断请求后`recv()`的操作,
如果得到response后还有后续操作，那么是有用的，否则并没有什么意义。想想可能是PHP操作粒度太大，
猜测用 C/C++ 应该能解决问题。

写的时候没有注意到一个问题，有些方式是返回值，有些直接打印了，这样不好，应该统一使用返回值得到请求结果。能力有限，先这样吧。

最后要做个广告，[计蒜客](http://www.jisuanke.com/)是一家致力于计算机科学高端教育的公司，如果你对编程或者计算机底层有兴趣，不妨去他们网站学习学习。\
同时，公司也一直在招人，如果你对自己的能力有信心，可以去试试。公司非常自由开放，90后为主。牛人也有不少，ACM世界冠军，知乎大牛。\
公司主做教育，内部学习资料必须给力，我只看到了一些关于操作系统的测试题，涉及到的知识面很广，可见公司平均技术能力有多厉害。

> 如果文章中有疏漏，错误，还请大神们不吝指出，帮助菜鸟进步，谢谢。

