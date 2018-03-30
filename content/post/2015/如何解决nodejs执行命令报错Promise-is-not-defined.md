---
date: 2015-10-07
title: 如何解决nodejs执行命令报错Promise is not defined
slug: nodejs-error-promise_is_not_defined
tags : 
- 问题
- NodeJs
topics: 
- 编程语言与开发
disqus_identifier: 100006
---

在使用c9.io时默认的node.js版本是v0.10.35，使得一些模块安装警告版本过低，而执行命令时报错。
Module build failed: ReferenceError: Promise is not defined

<!--more-->

### 问题表现

```bash
$ webpack
Hash: c978ec3fee3b9d59c3b7
Version: webpack 1.12.2
Time: 1263ms
    Asset     Size  Chunks             Chunk Names
bundle.js  9.04 kB       0  [emitted]  main
   [0] ./src/entiry.js 94 bytes {0} [built]
   [1] ./src/content.js 45 bytes {0} [built]
    + 3 hidden modules

ERROR in ./~/css-loader!./src/style.css
Module build failed: ReferenceError: Promise is not defined
    at LazyResult.async (/home/ubuntu/workspace/node_modules/css-loader/node_modules/postcss/lib/lazy-result.js:152:31)
    at LazyResult.then (/home/ubuntu/workspace/node_modules/css-loader/node_modules/postcss/lib/lazy-result.js:75:21)
    at processCss (/home/ubuntu/workspace/node_modules/css-loader/lib/processCss.js:174:5)
    at Object.module.exports (/home/ubuntu/workspace/node_modules/css-loader/lib/loader.js:22:2)
 @ ./src/style.css 4:14-76
```

### 问题原因
安装的node.js版本过低，而css-loader依赖的postcss插件最低node.js版本要求是0.12.故引起webpack执行报错。

### 如何解决

根本要升级node.js版本，那么如何升级nodejs呢

### 升级nodejs

nodejs非常活跃，更新频繁，我们可以直接使用nodejs的版本管理器`nvm`进行更新，同时可以非常方便的进行版本切换。

+ 安装nodejs版本管理器nvm（相比你机器已经安装了npm）
```shell
$ npm install -g nvm
```

+ 查看当前node的版本情况

```shell
$ nvm ls

	iojs-v1.7.1
		v0.6.21
		v0.8.28
->      v0.10.35
		v0.11.14 
		 system
default -> 0.10.35 (-> v0.10.35)
node -> stable (-> v0.12.7) (default)
stable -> 0.12 (-> v0.12.7) (default)
unstable -> 0.11 (-> v0.11.14) (default)
iojs -> iojs-v1.7 (-> iojs-v1.7.1) (default)
```

此时，我们看到本地nodejs版本最高是v0.11.14,环境使用的版本是v0.10.35.而非v0.12.7,故需要先下载最新版本后，再切换环境版本。

+ 下载指定nodejs版本

```shell
$ nvm install 0.12.7
######################################################################## 100.0%
Now using node v0.12.7
laoyu@nodestudy:~/workspace $ nvm ls
    iojs-v1.7.1
        v0.6.21
        v0.8.28
       v0.10.35
       v0.11.14
->      v0.12.7
         system
default -> 0.10.35 (-> v0.10.35)
node -> stable (-> v0.12.7) (default)
stable -> 0.12 (-> v0.12.7) (default)
unstable -> 0.11 (-> v0.11.14) (default)
iojs -> iojs-v1.7 (-> iojs-v1.7.1) (default)

```

通过nvm可以下载安装不同nodejs版本，`0.12.7`即为版本号。如果想下载最新版本，可以执行如下命令来获取。

```shell
$ nvm install stable
```

+ 切换环境中的node版本

    实际上在使用`nvm install`命令时便已切换环境node版本，如上一步便可知道。但有时候你需要切换环境node版本时，便可使用下面命令来切换。
```shell
$ nvm use 0.12.7
```
上面就是是环境版本切换到刚下载的版本0.12.7.


### 检查新node版本是否工作正常
对于前面出的的问题`Promise is not defined`，我们已经更新node版本，此时便可运行测试
```shell
$ webpack
Hash: 12fe1da0f27e13a0ff13
Version: webpack 1.12.2
Time: 1410ms
    Asset     Size  Chunks             Chunk Names
bundle.js  10.7 kB       0  [emitted]  main
   [0] ./src/entiry.js 94 bytes {0} [built]
   [1] ./src/content.js 45 bytes {0} [built]
    + 4 hidden modules
```
执行成功！

到此我们要解决此问题，其实还是环境问题，本文的目的是在于记录如何更新node版本以及进行版本切换。