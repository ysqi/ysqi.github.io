
---
date: 2017-02-17T08:17:14+08:00
title: "使用Homebrew安装配置golang环境"
description: ""
disqus_identifier: 1487290634267804604
slug: "shi-yong-Homebrewan-zhuang-pei-zhi-golanghuan-jing"
source: "https://segmentfault.com/a/1190000008317179"
tags: 
- golang 
categories:
- 编程语言与开发
---

### 安装Homebrew

在[Homebrew](http://brew.sh/)复制安装命令，在控制台运行完成安装

### 安装golnag

    $ brew update && brew upgrade
    $ brew install go

### PATH配置

创建一个目录作为`gopath`,在目录创建三个目录`bin`、`src`、`pkg`

    $ cd ~
    $ vim .bash_profile

编辑`.bash_profile`文件并保存，文件内容如下

    export GOROOT=/usr/local/opt/go/libexec
    # GOPAT为上面创建的目录路径
    export GOPATH=/Users/deweixu/coding/Go/go_path
    export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

运行`source .bash_profile`使配置的`PATH`生效。

### 安装完成

运行`go env`查看安装效果：

    $ go env
    GOARCH="amd64"
    GOBIN=""
    GOEXE=""
    GOHOSTARCH="amd64"
    GOHOSTOS="darwin"
    GOOS="darwin"
    GOPATH="/Users/deweixu/coding/Go/go_path"
    GORACE=""
    GOROOT="/usr/local/opt/go/libexec"
    GOTOOLDIR="/usr/local/opt/go/libexec/pkg/tool/darwin_amd64"
    CC="clang"
    GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/q3/kxp92gk548z_y9pc1n3qsztw0000gn/T/go-build078494854=/tmp/go-build -gno-record-gcc-switches -fno-common"
    CXX="clang++"
    CGO_ENABLED="1"

`enjoy golang`

