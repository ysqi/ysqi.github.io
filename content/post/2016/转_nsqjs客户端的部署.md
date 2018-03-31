
---
date: 2016-12-31T11:33:35+08:00
title: "nsqjs客户端的部署"
description: ""
disqus_identifier: 1485833615310045449
slug: "nsqjske-hu-duan-de-bu-shu"
source: "https://segmentfault.com/a/1190000005156677"
tags: 
- centos 
- nsq 
- node.js 
- golang 
categories:
- 编程语言与开发
---

因为公司在业务中需要用到消息队列产品，我选用了基于golang开源的nsq产品，记录下我遇到的那些部署中的坑。\
首先安装nsq，这个没什么好说的，我是直接在[官网下载bin文件](http://nsq.io/deployment/installing.html)，直接部署的，环境是centOS
6.7，安装在/opt/nsq-0.3.7.linux-amd64.go1.6目录下；\
其次是安装nodejs，我安装的是v6.1.0版本，这步也没什么好讲；\
然后安装nsqjs这个遇到了些坑，这里先记录下\
1、要看下gcc的版本；

    $ gcc -v使用内建 specs。
    目标：x86_64-redhat-linux
    配置为：../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-languages=c,c++,objc,obj-c++,java,fortran,ada --enable-java-awt=gtk --disable-dssi --with-java-home=/usr/lib/jvm/java-1.5.0-gcj-1.5.0.0/jre --enable-libgcj-multifile --enable-java-maintainer-mode --with-ecj-jar=/usr/share/java/eclipse-ecj.jar --disable-libjava-multilib --with-ppl --with-cloog --with-tune=generic --with-arch_32=i686 --build=x86_64-redhat-linux
    线程模型：posix
    gcc 版本 4.4.7 20120313 (Red Hat 4.4.7-16) (GCC)

2、因为node.js
4升级了v8引擎，要求gcc的版本在4.8以上，所以要先更新gcc版本；

    $ rpm -ivh https://www.softwarecollections.org/en/scls/rhscl/devtoolset-3/epel-6-x86_64/download/rhscl-devtoolset-3-epel-6-x86_64.noarch.rpm
    $ yum install devtoolset-3-gcc-c++
    临时使用最新版gcc：
    $ scl enable devtoolset-3 bash
    系统默认使用gcc-4.9
    $ echo "source /opt/rh/devtoolset-3/enable" >>/etc/profile

3、然后安装nsqjs，为了项目的复用，我就用了全局安装，然后把nsqjs复制到项目的node\_modules中就可以了；

    $ npm install -g nsqjs

4、把nsqjs复制到项目的node\_modules目录下；\
5、在项目中建立个app.js文件，输入以下代码并保存：

    var nsq = require('nsqjs');

    var reader = new nsq.Reader('sys_topic', 'sys_chan', {
        lookupdHTTPAddresses: '127.0.0.1:4161'
    });

    reader.connect();

    reader.on('message', function(msg) {
        var t = new Date();
        console.log('time [%s] Received message [%s]: %s', t.Format('yyyy-MM-dd hh:mm:ss'), msg.id, msg.body.toString());
        msg.finish();
    });

    Date.prototype.Format = function(fmt) { //author: meizz 
        var o = {
            "M+": this.getMonth() + 1, //月份 
            "d+": this.getDate(), //日 
            "h+": this.getHours(), //小时 
            "m+": this.getMinutes(), //分 
            "s+": this.getSeconds(), //秒 
            "q+": Math.floor((this.getMonth() + 3) / 3), //季度 
            "S": this.getMilliseconds() //毫秒 
        };
        if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
        for (var k in o)
            if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
        return fmt;
    }

6、在nsqadmin的页面中，创建Topic为“sys\_topic”和channel为“sys\_chan”；\
\
7、在应用终端中，运行这个js文件

    $ node app.js

8、在另外一个终端中发布一个消息

    $ curl -d '{"aa":"text","caption":"nsq_test","bool_v":true}'  'http://127.0.0.1:4151/put?topic=sys_topic&channel=sys_chan'

9、看看到我们能非常快的接收到发布的消息



