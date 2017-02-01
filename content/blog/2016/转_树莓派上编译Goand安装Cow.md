
---
date: 2016-12-31T11:35:02+08:00
title: "树莓派上编译Goand安装Cow"
description: ""
disqus_identifier: 1485833702052356685
slug: "shu-mei-pa-shang-bian-yi--Go-and-an-zhuang--Cow"
source: "https://segmentfault.com/a/1190000000456312"
tags: 
- raspberry-pi 
- golang 
- httproxy 
topics:
- 编程语言与开发
---

*PS:老Blog文章转移, 年代久远, 连接可能已失效.*

Cow
是不错的软件，相当好用，我在公司是直接把它挂到了服务器上，然后办公室的人都在用它。但是回到了家里我就无法用移动设备或者
PSP 之类的连接它了，我的电脑也不能一天 24
小时的在家中开机，碰巧这两天买了连个树莓派，上面运行的是专门定制过的
Debian Linux，我就想着是否能够使用它来运行
Cow。十分不幸的是，似乎作者的网站上并没有提供 ARM 设备的 Cow
版本，我尝试的下载了 Linux 32 位的版本，但是无法在树莓派上运行。所幸 Cow
是用 Go 语言写的，而 Go 支持 ARM，大不了自己编译 Cow。

我不知道是因为我的的问题还是因为什么奇怪的问题，树莓派的官方源里有
Golang，但是我安装之后却无法使用。So，干脆连 Go 也自己编译好了。

先安装依赖包：

`sudo apt-get install -y mercurial gcc libc6-dev`

然后用 Mercurial 拖回 Go 的源码：

`hg clone -u default https://code.google.com/p/go $HOME/go`

然后开始编译:

`cd $HOME/go/src ./all.bash`

这一步非常非常非常漫长，我估计我等了能有七八十分钟。等待漫长的编译结束后，我们还需要设置一下环境变量，在`.zshrc`或`.bashrc`下加入`export PATH=$PATH:$HOME/go/bin`。然后重启
Shell
环境，执行一下`go version`命令，如果出现正确的版本号信息，就表示一切都
OK 了。如果你准备马上开始编译
Cow，还需要设置一下`gopath`，在`.zshrc`或`.bashrc`中加入`export GOPATH=$HOME/mygo`，然后执行`go get github.com/cyfdecyf/cow`命令开始拖回
Cow 的源码并编译。

又是一阵漫长的等待，之后 Cow
的可执行文件会出现在\$HOME/mygo/bin/目录之中，最后附我所编译好了的 Go
for Raspberry pi 与 Cow for Raspberry pi 下载地址与 Cow 项目主页：

-   Go for Raspberry pi
    <http://pan.baidu.com/share/link?shareid=3899103835&uk=235347055>
-   Cow 0.7.1 for Raspberry pi
    <http://pan.baidu.com/share/link?shareid=3925804000&uk=235347055>
-   Cow 项目主页 <https://github.com/cyfdecyf/cow>


