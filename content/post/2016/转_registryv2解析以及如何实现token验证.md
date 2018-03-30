
---
date: 2016-12-31T11:34:10+08:00
title: "registryv2解析以及如何实现token验证"
description: ""
disqus_identifier: 1485833650071564189
slug: "registry-v2-jie-xi-yi-ji-ru-he-shi-xian-tokenyan-zheng"
source: "https://segmentfault.com/a/1190000003963021"
tags: 
- golang 
topics:
- 编程语言与开发
---

提到registry
v2，主要改进是支持并行pull镜像，镜像层id变成唯一的，解决同一个tag可能对应多个镜像的问题等等。如果还不太了解，可以且听我细细道来。

首先不得不说的是v2 新加了一个概念Digest
---------------------------------------

他是基于内容进行寻址（Content-addressable)算法算出来的一串hash值。简单的说就是内容不同，得出了的digest值是不同的，但是内容相同的话，得出的digest值是一定相同的。我们的每个镜像层id就是根据每个镜像层的内容得出来的digest的。

所以你在改动镜像层以后生成的digest就不同了，而不动的话，这个digest还是不变的，那么这个digest
id是什么时候生成的呢？我们在本地构建镜像时生成的镜像层id每次都是不一样的，这个digest是我们在push镜像时生成的。

为了验证内容相同，push到registry得到的digest相同，我做了个小实验，用如下Dockerfile
注释掉第三行和不注释构建了两次镜像，再push到registry

如果是v1的话，push上去得到的层id肯定是不一样的，但是v2里面，注释第三行得到了5个镜像层，不注释掉第三行得到了6个镜像层，并且第一次的5个层都包含在第二次6个里面。所以得出结论这个digest确实是根据内容生成的。

接下来说一下镜像的id
--------------------

镜像id也是生成了一个digest值，镜像id是根据\_manifest这个文件，也就是镜像层id和镜像名字等一些其他信息生成的digest。我们在每次向v2
push镜像时候，最后都会返回给docker
client一个digest值，这个值就代表了镜像的digest
id。这个id的作用就是可以指定唯一的镜像了。类似tag使用。

因为我们知道v1时候用tag有个弊病就是多次构建的镜像可以使用同一个tag，导致我们用tag标识镜像的时候可能并不是我们想要的，而用了digest就不会出现这种问题。

我们在写Dockerfile的时候引用镜像就可以这么用：

FROM
localhost:5000/test@sha256:ac81211548c0d228e10daaf76f6e0024e5f91879c8a7e105e777d6f964270449

像使用tag一样，用本地docker查看镜像digests时候可以使用：docker images
--digests，

当然，目前来说你看到的都是&lt;none&gt;，我们需要从registry上pull下来，使用\
docker pull
localhost:5000/test@sha256:ac81211548c0d228e10daaf76f6e0024e5f91879c8a7e105e777d6f964270449

了解了digest，我们来看一看registry的存储结构
--------------------------------------------

这部分最好可以对着registry的文件夹结构来看。简单的画了个草图。

是v2的文件夹层级关系

这个是目录下具体的样子，可以先看一会儿，可以看到registry
下面有两个文件夹blobs和repositories，

blobs下面存储了registry的所有基本信息元素，包括镜像层digest和镜像digest，之后在通过某种方式将调用这里的信息。

blobs文件夹下面先分了一个等级，是digest的前两位字符组成的文件夹为了筛选digest，避免了这个文件夹太大，查看时都难。这样方便定位。每个blob里都有一个data文件，存储相关信息。

repositories下面存储的是各个镜像名字命名的文件夹，进入一个镜像文件夹后，可以看到三个子文件夹，\_layers,
\_manifests,
\_uploads，\_layers这个文件夹是跟这个镜像有关的所有镜像层，进入其中一个镜像层文件夹，下面只有一个文件--link命名的文件，里面的内容就是一个digest。

这就跟blobs下面的镜像层建立起了联系，在repositories这个文件夹下，都是通过link文件与blobs文件夹下的文件建立的联系。

\_upload这个文件夹，平时点进去是空的，这个文件夹主要作用是上传用的。我们上传镜像层的时候，先上传到这个文件夹下，等完成传输以后，在将这个文件夹下的内容移动到blobs下面，然后将原来的文件删除。

\_manifest这个文件夹非常重要，是镜像的相关信息。他下面有两个子文件夹，revisions和tags，revisions这个文件夹下是这个镜像的所有可用的镜像digest，里面的link文件指向了镜像的digest。我们去blobs里面找相应的id对应的文件，查看文件下面的data，我们发现这个data文件里面存储的信息，和我们通过registry
v2 rest
api请求manifest信息相同\~在看\_manifest/tags/。下面就是这个镜像的不同tag了。又分了current和index分别表示当前tag对应的digest和此tag下的所有镜像digest。

下面来介绍一下如何搭建token验证的registry
-----------------------------------------

先看一下官方给的图

可以看到，v2和v1相比，访问形式完全变了，去掉了index
server，加上了一个auth server。

访问顺序是这样的

1.  我们通过docker client让docker
    deamon先访问registry，registry如果不需要身份验证，则直接返回结果，若需要验证，返回401并在header附带一些信息，

2.  daemon根据信息访问auth server。auth
    server判断通过了验证，并返回给daemon一串token。

3.  daemon带着这串token再去访问registry则可以获得到信息，pull
    ，push，api都走的这套流程。

接下来贴下我的配置文件config.yml，配置了选项auth/token表示要token验证。这个流程确实需要好好读一下，跟她的加密方式有很大关系

version: 0.1\
log:\
fields:\
service: registry\
storage:\
cache:\
blobdescriptor: redis\
filesystem:\
rootdirectory: /var/lib/registry\
http:\
addr: :5000\
secret: randomstringsecret\
tls:\
certificate: /root/sslkeys/domain.crt\
key: /root/sslkeys/domain.key\
auth:\
token:\
realm: <https://registry.tenxcloud.com>:5001/auth\
service: test123.tenxcloud.com:5000\
issuer: qwertyui\
rootcertbundle: /root/sslkeys/domain.crt\
当然，我是通过容器启动的v2，把这个config volume进去的

我的启动命令：docker run -d -p 5000:5000 --restart=always --name
registry -v `pwd`/sslkeys:/root/sslkeys -v
`pwd`/config.yml:/etc/docker/registry/config.yml -v
`pwd`/data:/var/lib/registry registry:2.1.1

auth/token下的4个子选项都是必须配置的，realm表示我的auth
server地址，service表示的registry的地址，issuer是一串标示符，随便写一下，auth
server加密的时候也需要配置同样的字符串。rootcertbundle配置一个秘钥，对token进行加密。（我这里复用了我的tls
token）

当我们第一次未带token访问registry时会返回401并附带如下信息：Www-Authenticate:
Bearer realm="test123.tenxcloud.com:5000
",service="test123.tenxcloud.com:5000
"scope="repository:husseingalal/hello:push"，realm和service就是前面说的，scope表示的是我要做的操作，repository代表镜像，接着是镜像名字，然后是pull或者是push或者二者都有

知道了各个参数的功能，就可以搭建我们的auth
server了，我从github下找到了一个项目：<https://github.com/cesanta/docker_auth>

用go语言写的，其中的账号密码存储方式有：写入文件，ldap和google账号的。这个server实现了动态加载配置文件，配置文件有变化这个server会进行安全的重启，所以可以对文件动态添加账号密码。当然也可以自己写身份验证，添加数据库等方式的，只需要继承Authenticator这个interface就可以。添加起来很容易。身份验证后有权限控制acl，并且我们也可以自定义权限控制，继承Authorizer这个interface即可。

前两项通过后就是生成token（这一部分这个项目已经封装好了，不用改动），\
v2的token采用的JWT加密方式，JWT分了三个部分，header，payload,signature，header里面带的信息是token加密方式，一般都是base64，
payload里带的都是需要的基本信息，user,权限，过期时间，还有前面说的issure
等等，\
signature是header+payload后用秘钥进行加密，这个秘钥就是配置在registry里的rootcertbundle对应的秘钥。

三部分通过 . 连接,
因为有秘钥加密，保证了token信息不会被窜改，这种加密方式保证了将权限验证和registry分离也
是安全的,
将token发送回daemon后daemon就可以带着token去正常访问registry了，如果是rest
api，直接将其写成Bearer token就可以请求了。

本文作者:时速云工程师丁麒伟 原文链接：[registry v2
解析以及如何实现token验证](http://blog.tenxcloud.com/?p=951)

