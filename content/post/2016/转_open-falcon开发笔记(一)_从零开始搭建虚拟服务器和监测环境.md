
---
date: 2016-12-31T11:32:45+08:00
title: "open-falcon开发笔记(一):从零开始搭建虚拟服务器和监测环境"
description: ""
disqus_identifier: 1485833565646767148
slug: "open-falcon-kai-fa-bi-ji-(yi-):cong-ling-kai-shi-da-jian-xu-ni-fu-wu-qi-he-jian-ce-huan-jing"
source: "https://segmentfault.com/a/1190000007699521"
tags: 
- virtualenv 
- linux 
- python 
- golang 
categories:
- 编程语言与开发
---

收到新的任务研究一下 open-falcon
监控公司的服务器指标玩玩。说实话现在连要重视哪些监控指标都不知道，但在\
[open-falcon 中文介绍](http://book.open-falcon.org/zh/intro/index.html)
中，就安装这一块，踩的坑实在好多，所以有必要写篇文章记一下。\
这篇文章主要包括：

-   如何搭一个虚拟的 linux 服务器和相关配置

-   按照官方的中文介绍安装，会踩到哪些坑。

目标只有一个：尽快跑起来。

### 服务器选择

笔者最终选择的是 [ubuntu-server 16.04 LTS
64bit](http://www.ubuntu.org.cn/download/server)。基于这款服务器，许多安装指令和介绍中不一样。具体包括

-   yum 替换为 apt-get

-   mysql-devel 替换为 libmysqld-dev,libmysqlclient-dev

等。并且 ubuntu 的源需要配置。笔者配置的是阿里云的ubuntu 16.04 源，参考
[ubuntu 16.04
国内快速更新源](http://www.linuxidc.com/Linux/2016-06/132518.htm)\
这里有几点需要注意。

-   open-falcon 需要 64
    位的操作系统以实现快速安装，即便是源码安装也需要你手动调整命令区别。非熟悉者别擅自尝试。

-   尽量别选择 desktop 的操作系统。卡的要死。

-   安装之前会有一坨提示，别随便从电脑前走开。会问你要不要安装一些网络服务包，记得
    openssh 要装，到时用什么 Xshell 啊，Powershell
    啊去远程连接，支持文本复制到命令行和 vim
    什么的还是很好的。当然了你到时手动装也行。

虚拟机选择的是Virtual Box for Windows。分配了 2G 内存和 8G
硬盘（应该够用了吧 QAQ
）。第一次启动会提示你装载ISO。之后需要配置设备-网络-高级-端口转发。以及可能需要从主机传文件过去，一会讲。

从 VirtualBox 中配置 ssh 的 22
端口转发。记得在主机网络连接里查看主机网络，在虚拟机中使用 ifconfig
等命令查看子系统网络。显示命令不存在的话是缺相应的包了。如果子系统网络只有
ipv6 的话。。我还真不知道怎么办。估计自己用 ifconfig 命令重设一下 ipv4
地址？

#### CentOS 里的坑

这里说一下我一开始为了适配 yum 命令，选择的是 CentOS 7 Minimal
ISO。这货下载快安装也快，但进去之后要自己[配虚拟网卡](http://www.07net01.com/2016/01/1140061.html)。勾选什么的用空格。还有配网卡之前不推荐参见某些教程手撕
eth0 配置文件，网卡坏了改都改不过来。

默认的网卡名称还不是 eth0，而是enp0s3 ，似乎是 ipv6
连接的，这个会在之后端口转发的时候有影响。

不过最惨的坑还是CentOS 7 没有 mysql，excuse me?
啊好吧是被换成了完全向前兼容的子项目 [mariadb](https://mariadb.org/)
不过既然兼容 mysql 命令就先用上吧。\
在安完数据库之后照常进行[配置数据库远程访问](http://blog.csdn.net/preterhuman_peak/article/details/40396873)以通过数据库管理软件诸如
[Navicat](https://www.navicat.com.cn/store/navicat-premium)
（注：不便宜）进行访问。但是在配置完成之后会发现报错：

    SQL Error (2013): Lost connection to MySQL server at 'waiting for initial communication packet', system error: 0

/(ㄒoㄒ)/\~\~

搜这个问题会有一些提示你在 mysql 的配置文件 `my.inf` 或 `my.ini`当中在
\[mysqld\] 配置项加上\
skip-name-resolve 的，有让你删掉 bind-address=127.0.0.1
的（这个的确是必要的），还有让你重启数据库连接的?\_?。\
照做的时候发现问题了。mariadb
的配置文件是采用导入别的文件夹里的配置的方式的。由于水平所限不了解 mysql
配置体系，于是这个坑回来再填。\
如果有小伙伴知道这里怎么解决还请赐教。

只好弃坑 mariadb 去重新用了 ubuntu server。到这里其实半天已经没了 QAQ。

### 配置准备环境

redis 不一定存在这个包，可以尝试换成 redis-server。\
创建工作目录恐怕是必要的，保不准到时候那些脚本会不会直接调用这个 export
变量。\
配置 mysql 如果要用远程连接记得去配置项里面把 bind-address
这一行注释掉，否则不能远程访问的。

准备环境里有要求下载已经编译好的 `of-release-v0.1.0.tar`
组件的，这地方就有坑了。从 github
下载这个包的确巨慢，只好自己手配主机到虚拟机的传送环境。一共是三种方法：

-   [利用 VirtualBox
    的共享功能（需要安装增强功能）](http://www.cnblogs.com/xing901022/p/5774677.html)

-   [scp （注意 Windows
    下需要额外安装包）](http://www.tuicool.com/articles/EJjIBr)

-   [虚拟机搭 FTP
    服务器：vsftpd](http://www.cnblogs.com/zemliu/archive/2012/06/07/2539135.html)

其中第一个需要 VirtualBox 的增强功能，我在 windows
上面因为缺少安装盘报了错；第二个未验证；第三个记得配置服务器的读写权限。

不厌其烦地提醒一下下好了之后解压，命令循环解压，若不是 root 用户，是需要
sudo 的。安装好之后你的文件夹里应该是这个状态：\

接下来就是逐个根据教程安装服务了。

### 服务安装的注意事项

在绘图组件当中，graph 组件是要连接数据库的，并且设置 root
的密码为空的连接会失败。表现为查看`./control tail`
时会发现启动失败，报了Access Denied for root[@localhost](/u/xingdong365)
的类似错误。但比较坑的是在启动的时候咋不给我报出来呢，启动显示成功查进程查不到看
log 又只有这么点信息，因为对 golang 不熟也没能跟踪原因具体是啥。查看
mysql 的 user 表会发现无密码的登录方式是 plugin:auth\_socket，而其他的是
mysql\_native\_password + authentication\_string。至此萌比。\
总之我是修改用户密码来解决问题的，为此要变更`$WORKPLACE/graph/cfg.json`：

    "db": {
            "dsn": "root:yourpassword@tcp(127.0.0.1:3306)/graph?loc=Local&parseTime=true",
            "maxIdle": 4 
        },

一改这个配置其他服务组件全都要改，其实这并不是个优雅的解决方案。\
如果有小伙伴了解为何不用密码连接会失败，比如看个源码啥的，请告诉我谢谢。（我可能以后会补。）

dashboard 组件是基于 python virtualenv 去搭建的。mysql-devel 会被替换为
libmysqld-dev，除此之外还需要安装
python-dev/python2.7-dev（取决于你的版本，应该知道 python 2 和 python 3
有很大不兼容性。）,否则下一步 pip install requirements
的时候就会发现没有 gcc 去编译一些依赖包。

完事之后把服务一个个启动起来就行了。查 log
的时候会报错此时还正常，毕竟告警组件还有一堆没安装。

启动之后打开表盘，左边输入你的服务器机器名（ubuntu），左边找出来之后再到右边点查找。screen
里面是空的应该没有各种统计指标啥的。不要吓着恩。

就先写到这里。接下来会有关于告警组件的坑，还有业务相关什么的。

