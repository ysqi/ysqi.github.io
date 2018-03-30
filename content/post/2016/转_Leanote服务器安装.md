
---
date: 2016-12-31T11:34:07+08:00
title: "Leanote服务器安装"
description: ""
disqus_identifier: 1485833647159276130
slug: "Leanote-fu-wu-qi-an-zhuang"
source: "https://segmentfault.com/a/1190000004000321"
tags: 
- 开放源代码 
- evernote 
- golang 
- leanote服务器 
- leanote 
topics:
- 编程语言与开发
---

我之前用过很多笔记产品, 比如evernote, 有道, 为知, oneNote.
一直想寻找一个简单好用, 能集成博客功能的笔记.

一直找了好久, 终于有一天, 找到了Leanote, Leanote简单好用, 有笔记, 博客,
分享功能. 功能简单好用恰到好处. 竟然还开源, 看到时, 眼前一亮,
这么多的产品, 已经足够产品化, 有桌面端, ios端. 竟然还开源.
让我更加惊讶的是, 这是国我开发的....

不说了, 反正非常兴奋. 开源的话, 那肯定可以自己安装到本地,
成为私有的云笔记.

下面我就来说说怎么安装Leanote啦.

其实也就是参考了官方wiki <https://github.com/leanote/leanote/wiki,>
没什么特别的:

-   [leanote binary distribution installation
    tutorial](https://github.com/leanote/leanote/wiki/leanote-binary-distribution-installation-tutorial)

-   [leanote develop distribution installation
    tutorial](https://github.com/leanote/leanote/wiki/leanote-develop-distribution-installation-tutorial)

-   [leanote二进制版详细安装教程](https://github.com/leanote/leanote/wiki/leanote%E4%BA%8C%E8%BF%9B%E5%88%B6%E7%89%88%E8%AF%A6%E7%BB%86%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B)

-   [leanote二进制版详细安装教程-Windows](https://github.com/leanote/leanote/wiki/Leanote%E4%BA%8C%E8%BF%9B%E5%88%B6%E7%89%88%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B---Windows)

-   [leanote开发版详细安装教程](https://github.com/leanote/leanote/wiki/leanote%E5%BC%80%E5%8F%91%E7%89%88%E8%AF%A6%E7%BB%86%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B)

-   [leanote开发版详细安装教程-Windows](https://github.com/leanote/leanote/wiki/Leanote-for-Windows-Setup)

------------------------------------------------------------------------

当时看到这多么链接就萌了, 安装个Leanote服务器还有这么多链接啊.
到底选哪个呢? 仔细研究之后, 发现其实两者, 二进制版和开发版.
二进制版就是已经编译好了的, 不用自己安装开发环境.
开发版就是需要安装开发环境, 给开发人员用. 像我这种技术小白,
还是不折腾开发版了. 老老实实安装二进制版省事. 但我之后也安装了开发版,
其实也简单.

我就安装二进制版了. 参考链接为:
[leanote二进制版详细安装教程](https://github.com/leanote/leanote/wiki/leanote%E4%BA%8C%E8%BF%9B%E5%88%B6%E7%89%88%E8%AF%A6%E7%BB%86%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B)

安装步骤:
---------

1.  下载leanote二进制版

2.  安装mongodb

3.  导入初始数据

4.  配置leanote

5.  运行leanote

下载leanote二进制版
-------------------

下载 [leanote 最新二进制版](http://leanote.org/#download)

自己选一个, 我自己用的linux 64位. 点击链接其实是跳到
<https://sourceforge.net,> 看来Leanote二进制版是发到这里.
<https://sourceforge.net/projects/leanote-bin/> 我在想,
为什么不把二进制版放在github上呢? 可能这里更方便吧.

把下载的文件下载到 \~/software 下, 解压文件

    $> cd ~/software
    $> tar -xzvf leanote-linux-amd64-v1.3.1.bin.tar.gz

安装mongodb
-----------

安装的:
<https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.0.1.tgz>

下载到\~/software 下, 直接解压即可

    $> cd /home/user1
    $> tar -xzvf mongodb-linux-x86_64-2.6.4.tgz/

添加到环境变量中\
编辑 /etc/profile 将mongodb bin路径加入.

    $> sudo vim /etc/profile
    添加:
    export PATH=$PATH:/home/alaege/mongodb-linux-x86_64-3.0.1/bin

使环境变量生效:

    $> source /etc/profile

### 简单使用mongodb

先在\~下新建一个目录data存放mongodb数据

    mkdir ~/data

    # 开启mongodb
    mongod --dbpath ~/data

这时mongod已经启动了

重新打开一个终端, 使用下mongodb

    $> mongo
    > show dbs
    ...数据库列表

mongodb安装到此为止, 下面为mongodb导入数据leanote初始数据

导入初始数据
------------

leanote初始数据在 \~/leanote/mongodb\_backup/leanote\_install\_data中

打开终端, 输入以下命令导入数据.

    mongorestore -h localhost -d leanote --dir ~/leanote/mongodb_backup/leanote_install_data/

现在在mongodb中已经新建了leanote数据库, 可用命令查看下leanote有多少张表

    $> mongo
    > show dbs #　查看数据库
    leanote    0.088125GB
    local    0.078125GB
    > use leanote # 切换到leanote
    switched to db leanote
    > show collections # 查看表
    files
    note_contents
    notes
    notebooks
    ....

初始数据users表中已有2个用户: 这两个用户供登录Leanote的,
demo用户是为了测试, admin用户特别重要. 因为只有admin用户才能管理后台.

    user1 username: admin, password: abc123 (管理员, 只有该用户才有权管理后台, 请及时修改密码)
    user2 username: demo@leanote.com, password: demo@leanote.com (仅供体验使用)

配置leanote
-----------

文件: conf/app.conf

修改app.secret, 随意修改一个值, 官方文档说不修改会安全问题, 管他呢,
随便改改就行.

运行leanote
-----------

**这里特别注意** 在此之前请确保mongodb已在运行!
所以不要用之后开启mongodb的窗口, 新开一个窗口吧!

新开一个窗口, 运行:

    $> cd ~/leanote/bin
    $> bash run.sh 
    # 最后出现以下信息证明运行成功
    ...
    TRACE 2013/06/06 15:01:27 watcher.go:72: Watching: /home/life/leanote/bin/src/github.com/leanote/leanote/conf/routes
    Go to /@tests to run the tests.
    Listening on :9000...

打开浏览器输入: `http://localhost:9000`

------------------------------------------------------------------------

这一路走来, 其实非常简单, 但作为小白的我, 也走了几个坑.\
1) admin用户名改了, 进不了后台管理了\
2) 数据库连不上啊, 提示 "no reachable server", 这可难倒我了.
后台把app.conf的mongodb地址改成了 127.0.0.1 就行了, 不知道为什么.
如果有大神知道, 就告诉我吧

其实所以坑基本上都在 <https://github.com/leanote/leanote/wiki/QA>
上提到了, 我也在这里找到了.

最容易犯的错就是用admin用户登录后, 把用户名改了.改了就悲剧了啊,
下次就不能进后台管理了. 悲剧. 怎么办?

其实很简单, 只要把conf/app.conf修改下,
把adminUsername=admin改成你改之后的用户名即可. 改完了还要重启leanote,
不然不生效. 当时没重启, 又搞了很久.

------------------------------------------------------------------------

还有一个问题是, 安装了Leanote服务后, Leanote也桌面端和ios端,
怎么连接到自己搭的服务呢? 这个Leanote的客户端做的很完善了,
在登录界面多看几眼, 试试就行

桌面客户端:

\
点击"self-hosted service"

第一行就填自己服务器的地址就行, 比如 <http://a.com>:9000,
没端口的去掉就行.

ios也是一样的:

要注意的是, 服务器地址在最后一行.

完美, 搞定.

------------------------------------------------------------------------

未完, 待续, 欢迎关注我的专栏, 关于使用, 安装Leanote的任何事,
我都会在这里发布. 太兴奋了.

