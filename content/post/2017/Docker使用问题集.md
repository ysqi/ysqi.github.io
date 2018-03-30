---
title: "Docker使用问题集"
date: 2017-10-11T07:11:07+08:00
draft: false
description: ""
slug: "use-docker-question-answer" 
tags:
- "Docker"
topics: 
- 编程语言与开发
---

接触Docker时间不长，折腾不少。实际上Docker还是蛮简单的，基于Go开发，容器化资源。

第一次创建镜像过程中遇到些问题，我在这里记录，希望能帮助到别人和自己。

## 国内如何加速拉取Docker镜像
通过 Docker 镜像加速，国内能够快速访问最流行的 Docker 镜像。国内用户现在将会享受到更快的下载速度和更强的稳定性，从而能够更敏捷地开发和交付 Docker 化应用。

+ [Docker官方提供的加速器](https://www.docker-cn.com/registry-mirror)
+ [DaoClund提供的加速器](https://www.daocloud.io/mirror)，需注册
+ [Aliyun 提供的加速器](https://cr.console.aliyun.com)，需注册，适合在阿里云购买的服务器使用。


## Docker-runc not installed on system
CentOS通过命令安装Docker时，并没有安装完整。一般在重新安装Docker后出现该问题。
只运行某镜像时，出现如下错误信息：
```shell
docker run docker.io/ysqi/gotestreport
/usr/bin/docker-current: Error response from daemon: shim error: docker-runc not installed on system.
```
咋回事？原本好好的怎么就出问题了？原来是命令卸载Docker并没有卸载干净，导致重装或更新Docker后出问题。

这个在红帽上有一个Issue：https://access.redhat.com/solutions/2876431 有说明此问题，暂时只能是新建一个link到最终版本：
```
cd /usr/libexec/docker/
ls -all
sudo ln -s docker-runc-current docker-runc 
```

## Jenkins: Can't connect to Docker daemon
当Jenkins的Pipeline基于Docker运行时，提示此错误信息：

> Cannot connect to the Docker daemon. Is the docker daemon running on this host?

这是因为CentOS安装Docker时使用的独立的用户组Docker，而Jenkins运行也是独立的用户Jenkins，需要将Jenkins用户添加到Docker的用户组中：
```shell
sudo gpasswd -a jenkins docker
```
再重启Docker服务
```shell
systemctl daemon-reload
systemctl restart docker
```
最后重启Jenkins
```shell
sudo service jenkins restart
```
Done！

## Docker: 如何将本地文件载入Docker容器中

运行Docker镜像时，是支持参数配置的。参数`-v`是表示映射本地目录到Docker容器中。
```text
-v, 
--volume list Bind mount a volume 
--volume-driver string Optional volume driver for the container 
--volumes-from list Mount volumes from the specified container(s) 预先设置将本机路径
```
例如：
```shell
export src=$HOME/goproject/ysqi/com
export target=/go/src/github.com/ysqi/com
```
再启动容器时设置挂载：
```shell
docker run -v $src:$target  ysqi/gotestreport
```
表示什么意思呢？即将本地的目录$src映射到Docker容器中的一个目录$target, 在Docker容器内部访问/go/src/github.com/ysqi/com，实际上就是在访问本机目录：$HOME/goproject/ysqi/com。

Docker实际上做了一个 volume 映射，就想VM上添加了一个映射盘。那如何把当前目录映射到Docker容器的指定位置？
```shell
docker run -v ${pwd}:/go/src/pkg ysqi/gotestreport
```
这样就把本地的当前目录，映射到了/go/src/pkg 目录。

## Docker: 如何在主机和容器间拷贝文件

上面所说得中使用-v 参数将主机与容器中相关目录联系在一起（挂载），所以我们可以用这个通道将想要互相拷贝的数据放入其中，这样就可以用 cp 命令来复制文件了。

除了这个办法，我们还可以分别用不同的命令来拷贝数据。
 
先`docker cp -h`查看命令使用
```shell
Flag shorthand -h has been deprecated, please use --help

Usage:	docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH
	    docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

Copy files/folders between a container and the local filesystem 
```
以Nginx为例：先运行Nginx容器：
```shell
docker run -p 8080:80 -d nginx
a54b77dbef541a7fce911b3298b32f4d4884665f88d2d23b690181a3dd684439
```
本机测试Nginx访问：`curl  http://localhost:8080`，正常。

再从主机拷贝文件到容器：
```shell
docker cp  ./test a54b77dbef54:/usr/share/nginx/html
```
此时将主机当前目录下的 test 文件夹全部拷贝到容器内部的 /usr/share/nginx/html目录下。

反之，则可将文件或文件夹从容器内部拷贝到主机指定目录：
```shell
docker cp  a54b77dbef54:/etc/nginx/conf.d/default.conf .
```
此时，便将容器内部的nginx默认配置文件拷贝的主机的当前目录下。