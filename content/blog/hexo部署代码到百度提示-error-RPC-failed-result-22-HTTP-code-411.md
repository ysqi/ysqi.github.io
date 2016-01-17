---
title: hexo部署代码到百度提示:error:RPC failed;result=22,HTTP code=411
slug: hexo-deploy-git-push-error-rpc-411
date: 2015-08-31T10:01:36
topics:
    - 软件
tags:
    - hexo
    - git
    - WebIDE
    - 百度
    - 问题
---

终于在 Coding.net 上部署搭建了写作环境,同时提交到我的[博客](http://www.superzhu.com),但在deploy到百度应用时有报错,在这里记录下解决方案.

## 问题表现

执行`hexo deploy` 时提示`error:RPC failed;result=22,HTTP code=411'`
具体错误信息见下
```bash
hexo deploy
error: RPC failed; result=22, HTTP code = 411
fatal: The remote end hung up unexpectedly
fatal: The remote end hung up unexpectedly
......
Everything up-to-date
```
## 错误原因

新搭建的环境,public 内容初始状态不是来源于 BAE git,而在首次 deploy 时提交的内容过大,而被 BAE 拒绝.

## 解决方案

修改hexo-doployer-git 下 git 的缓冲区大小,执行代码如下
```bash
cd .deploy_git/.git
vim config
```
再在文档中添加或修改配置项,修改到50M
```ini
[http]
    postBuffer = 52428800
```

再次执行`hexo deploy` 时成功,一键发布到 BAE!
