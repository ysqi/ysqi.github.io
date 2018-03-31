
---
date: 2016-12-31T11:33:16+08:00
title: "深入体验bashonwindows在windows上搭建原生的linux开发环境酷！"
description: ""
disqus_identifier: 1485833596368823479
slug: "shen-ru-ti-yan-bash-on-windowszai-windowsshang-da-jian-yuan-sheng-de-linuxkai-fa-huan-jing-ku-！"
source: "https://segmentfault.com/a/1190000006175018"
tags: 
- windows 
- node.js 
- golang 
- git 
categories:
- 编程语言与开发
---

今年微软Build
2016大会最让开发人员兴奋的消息之一，就是在Windows上可以原生运行Linux
bash，对开发人员来说，这是一个喜闻乐见的消息。

1 安装
------

你必须安装开发者预览版本，才能使用windows的linux subsystem功能。

1.  首先打开你的Windows
    10的设置，在"更新和安全"选项中（我的机器是英文操作系统，但中文操作系统类似），选择"开发者"，然后选择"开发者模式"：

2.  在Windows 更新和安全选项中，选择"Windows
    更新"，在"更新设置"中选择高级选项，你必须打开"内部预览版本"选项，并且将内部预览的级别设置为最快：

3.  在"设置"-&gt;"系统"选项中，查看当前系统信息，你的OS版本必须是14316，如果不是这个版本，请运行在线升级，升级到这个版本：

4.  升级完成后，打开"控制面板"-》"程序"，选择"打开或者关闭Windows特性"，找到Windows
    Subsystem for Linux(beta)点选并确定，系统会下载更新并提示重启机器：

5.  重启完成后，进入命令行程序，激动人心的时刻来临了，在命令行下敲bash，会提示你是否继续，选择yes，系统会安装Ubuntu文件系统等待完成后，系统会切换到你熟悉的Linux
    bash模式，试试你熟悉的ls等命令：

6.  打开Windows菜单，你会发现系统中多了一个Ubuntu的图标，这个就是bash on
    Ubuntu on Windows：

7.  简单测试一下python，perl等语言，发现已经装好了，当然，这个就是Ubuntu的bash，你可以安装Java，Ruby等等：

下面来实际测试下ubuntu on windows环境！

2 系统检测和更新
----------------

### 2.1 版本检测

运行 `uname -a` 结果如下

    webmaster@FEKETERIGO-PC:~$ uname -a
    Linux FEKETERIGO-PC 3.4.0+ #1 PREEMPT Thu Aug 1 17:06:05 CST 2013 x86_64 x86_64 x86_64 GNU/Linux
    webmaster@FEKETERIGO-PC:~$

运行 `lsb_release -a` 结果如下

    webmaster@FEKETERIGO-PC:~$ lsb_release -a
    No LSB modules are available.
    Distributor ID: Ubuntu
    Description:    Ubuntu 14.04.4 LTS
    Release:        14.04
    Codename:       trusty

可以看到，安装的是ubuntu 14.04 trusty

### 2.2 系统更新

既然是ubuntu，就可以使用apt-get进行软件包管理。

首先替换自带的更新源

    sudo mv /etc/apt/sources.list /etc/apt/sources.list.save

把下面的阿里源内容粘贴到 `/etc/apt/sources.list`

    deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse

    deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse

    deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse

    deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse

    deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse

    deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse

    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse

    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse

    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse

    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse

你也可以使用163源等，选取原则是更新速度，目前测试的情况是阿里云的源更新速度最快。

接着运行下面的命令进行系统更新，如果速度很慢的话请尝试使用其他的源，另外ubuntu的源是版本相关的，注意别添加错误版本的源。

    sudo apt-get update
    sudo apt-get upgrade

使用`sudo`命令之前可以运行 `passwd` 更新当前用户的命令，
bashonwindows默认运行在用户模式，\
windows会把当前用户添加到`sudoer`列表中，如果想切换到超级管理员，需要手动输入`su - `

3 搭建开发环境
--------------

### 3.1 使用apt和ppa repository安装git

ubuntu trusty默认的git版本是1.9.1

如果你不想使用git最新版本的话可以运行`sudo apt-get install git` 直接安装

如果想安装最新的git版本参考下面的命令

    sudo add-apt-repository ppa:git-core/ppa
    sudo apt-get update
    sudo apt-get install git

`sudo add-apt-repository ppa:git-core/ppa` 会在`/etc/apt/sources.list.d`
目录下面生成一个\
`git-core-ppa-trusty.list`文件，然后运行`sudo apt-get update`
的时候会从相应的仓库里面获取新\
的git版本信息。

由于每个人的网络环境不一样，
如果你更新失败，使用apt-get不能安装最新版本的git的话，请到github上面下载源码进行编译安装，过程也很简单\
这里就不写源码编译教程了， git源码点击
[这里](https://github.com/git/git)

### 3.2 使用pyenv搭建python开发环境

命令如下

    sudo apt-get install curl
    curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
    pyenv install 2.7.11
    pyenv install 3.5.1
    pyenv shell 3.5.1

python的2.x和3.x版本不兼容，所有把两个版本多安装到本地，方便需要的时候进行版本切换，使用命令`pyenv shell 2.7.11`\
指定当前的python版本。上面的命令指定的python版本是3

python的另一个有用的工具是virtualenv，它和pyenv都能实现python版本的切换，不过实现的方法却大相径庭。\
pyenv在用户模式下安装多个版本python，但是每一个python版本的包依赖还是安装的同一个位置，它不能解决两个项目中引用\
同一个库有版本冲突，但是viertualenv可以。

virtualenv给每一个python项目安装一个项目内部python，具体版本可以在初始化的时候指定，项目的依赖也是安装在内部python的\
路径下面，这样能防止和系统上其他python项目的依赖冲突，
隔离性更高，不好的地方是安装和使用略微复杂。

具体使用哪个工具看个人喜好，github上面pyenv的关注度要高点，而且现在pyenv有一个virtualenv插件，可以使用后者同样的功能，\
我个人还是推荐使用pyenv。

### 3.3 使用rvm安装ruby

    curl -L https://get.rvm.io | bash -s stable --autolibs=enabled --ruby
    rvm install 2.3.1
    bash --login
    rvm use ruby-2.3.1

具体命令输入`rvm help`，很好掌握。

### 3.4 使用gvm安装go语言

    curl -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer | bash
    gvm install go1.6
    gvm use go1.6 --default

由于[hugo](http://gohugo.io)是使用go语言开发的，趁着这个机会向学习一些这个由google开发的编程语言，本博客就是使用\
hugo搭建的，服务器使用[firebase](https://firebase.google.com/)，firebase提供和github
pages一样的免费静态服\
务器的功能，稳定性比github要好，github在国内的访问速度实在不能再慢了，特别是在clone大一点的项目的时候。

go语言开发的程序有一个好处就是没有运行时依赖，打包成exe就能直接运行，简单方便，更多的内容以后再更新吧，这里集中在开发环境\
搭建这个主题上面。

### 3.5 使用nvm安装nodejs

    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.30.0/install.sh | bash
    nvm install v6.2.0
    nvm use v6.2.0

`node.js`可以说是从2015年以来最火的语言了，node.js是后端工程师转向前端最好的工具，博主一起就是java后端开发工程师，偶然的机会\
学习了node.js，然后开始了自己的前端工程师的进化之路，目标是全栈工程师。

node.js还有一个npm包`n`可以用来进行版本管理，不过你需要先安装一个node.js才能使用`npm install -g n`安装这个工具

`n`和`nvm`都很好用，你可以随便选择一个，也可以两个都安装，他们之间没有冲突，可以共存。

> 这里注意一下，如果你使用的是 `windows build 14352`
> 的话，使用nvm安装nodejs可能会出现错误，详情参见[这里](https://github.com/Microsoft/BashOnWindows/issues/426)\
> 我安装的时候是出现问题，但是又没有看到其他人遇到这个问题，如果你安装的windows预览版是比14352更新的版本的话，你可以自己测试下

### 3.6 安装gcc工具链

    sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev

这些工具不仅仅是c和c++开发者才会用到，如果你开发一个node.js
native模块，你也需要这些工具。

### 3.7 安装nginx

    sudo apt-get install python-software-properties
    sudo add-apt-repository ppa:nginx/stable
    sudo apt-get update
    sudo apt-get install nginx

和安装git的时候一样，这里通过添加nginx的第三方repository，实现apt-get方式安装nginx最新版本，比源码编译安装方式在管理上还是要方便一点。

### 3.8 使用shell安装rust-lang

安装命令,非常简单

    curl -sSf https://static.rust-lang.org/rustup.sh | sh

rust是和node.js一样热门的新星编程语言，不过和node.js不同，rust不是一款前端领域的编程语言，而是一款系统编程语言。

rust的有点是安全、高并发，吸收了大量现代高级编程语言的有点，极力避免现有编程语言的各种缺点，让我印象最深刻的一个特性是\
**rust的垃圾回收机制**,

最开始的时候rust也是使用类似java语言的garbage
collector方式实现垃圾回收，后来受不了gc的\
运行时性能损耗，自己实现了一套更高效的垃圾回收机制，极大的提高的系统稳定性和性能。

而且在rust在1.0的时候已经实现了自举（自己编译自己）,后面版本的rust都是使用rust自己编译出来的，包括编译器。

唯一不好的地方是rust语言为了最大程度的帮助开发人员避免犯错误，采用了极其复杂的语法，对于刚开始学习编程语言的人来说，不建议学习\
rust，因为rust的学习曲线非常陡峭，入门语言选择c或者java都是不错的。

4 最后总结
----------

虽然这篇文章没有讲一些高深的内容（水平有限）， 但是还是总结一些。

### 4.1 关于包管理

不管是操作系统，linux、mac、windows，还是编程语言，java、node.js、rust，流行的趋势是**自带包管理工具**，

linux
有apt，centos有yum，mac有brew，windows目前没有，不排除以后不会有，win10应用商店已经有了，估计应用\
管理工具也不好很远了吧，目前已经有一个第三方的10AppsManager了。

java没有自带的包管理工具，但是`maven`，node.js有npm，rust有cargo。

使用包管理工具能帮助开发者解决很多的问题，例如版本管理，依赖管理，甚至软件发布都可以使用包管理工具来完成，node.jsd的包\
就是使用npm进行发布的。

所以我觉得**自带包管理工具**应该是以后编程语言的趋势，这方面做得最差的是java，从make到ant到ivy再到maven再到gradle，java的包\
管理是最复杂的，如果是心学习java的推荐学习gradle，gradle还能用来打包android项目，是目前最优秀的工具。

另外操作系统的包管理工具比编程语言又更悠久的历史，几乎每一个类linux操作系统都有自己的包管理工具。

### 4.2 版本管理

**软件版本管理**

每一个软件都会进行版本迭代，有时候进行版本更新的时候又会有**broken
update**，为了方便实现版本切换，现在很多编程语言\
都提供了版本管理工具（第三方的），所以我在想能不能把这个功能添加在编程语言上面，简而言之就是编程语言能自带一个版本管理\
工具（就像自带包管理工具一样），这样能更快的实现版本切换。

### 4.3 以开发人员为中心

随着windows开始各种拉拢[开发人员](https://lanmaowz.com/learn-bash-on-windows/)和软件工程师队伍的壮大，我认为**以开发人员为中心的时代，广大开发人员的春天就要到来了**，

