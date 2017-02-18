---
date: 2016-03-10T14:21:31+08:00
description: ""
disqus_identifier: "20160302"
slug: "wercker-deploy-github-qa"
tags:
- Go
- Wercker
- git
- github
title: Wecrcker自动部署到Github过程中问题记录
topics:
- 编程语言与开发
---

TMD总算顺利的将项目自动发布github中，这里分享下利用 wercker 自动部署到 github 时遇到的一系列问题。

### wercker 配置

```yaml
deploy:
  box: samueldebruyn/debian-git
  steps:
    - ysqi/git-push:
        gh_oauth: $GIT_TOKEN
        basedir: public
        clean_removed_files: true
        branch: $GIT_BRANCH
        repo: $GIT_REPO
        gh_pages_domain: $GIT_DOMAIN
```
又麻利又大方地分享下我的设置：

![wercker 环境变量](https://static.yushuangqi.com//blog20160310171328.png)

+ __box__ 
是必须要的，并且要包含 git 环境，尝试了各种 box 最终挑选了 samueldebruyn/debian-git 满足要求。

+ __ysqi/git-push __  
step 使用我自己的 git-push ，原来鼻祖的**leipert/git-push** 有问题，报错信息“getAllStepVars: command not found”
```
source "/pipeline/git-push-36805875-502f-4737-a412-1b6f6880f7a4/run.sh" < /dev/null
/pipeline/git-push-36805875-502f-4737-a412-1b6f6880f7a4/run.sh: line 5: getAllStepVars: command not found
using github repo "ysqi.github.io"
remote URL will be https://oauth-token@github.com/ysqi.github.io.git
```
这鼻祖没法玩，硬是不合并别人的PR，没法办只能自己动手，我也Fork了一份好好先生的。你也可以用我的 git-push

+ __$GIT_TOKEN__ 
是在 github 添加 token 后将生成的token 保存在 wercker 环境变量中，主要不要直接写在配置文件中，防止泄漏。

__先在 github 上生成 token：__[去设置](https://github.com/settings/tokens/new)
![在 github 上生成 token](https://static.yushuangqi.com//blog20160310155407.png)

__再在 wecrcker 上添加环境变量 GIT_TOKEN__:
![在 wecrcker 上添加环境变量](https://static.yushuangqi.com//blog20160310155210.png)
主要勾选 proteced 这样才能更严格的保护Token


+ __$GIT_BRANCH__   
这是部署到哪个分支，这个容易处理，总算可以理解。我配置 master

+ __$GIT_REPO__  
妈蛋，这就这样玩意害死我了，刚开始以为已经是 token 访问我的github账户，认为不需要知道全路径。便配置成`GIT_PEPO= ysq.github.io` 这个导致 git-push 执行失败，也不知错误原因，反正就显示：
```bash
source "/pipeline/git-push-a588ef90-1dd5-49c6-ad00-dd51ae73ccfb/run.sh" < /dev/null
using github repo "ysqi.github.io"
remote URL will be https://oauth-token@github.com/ysqi.github.io.git
```
咦！！！ remote url 是错误的！！ 缺用户名。应该是：`ysqi/ysqi.github.io` 。我操，修改后问题解决。

成功 git-push !!!
```bash
pushed to master on https://oauth-token@github.com/ysqi/ysqi.github.io.git
```

上面就是我解决 wecrker 过程中的问题记录，感兴趣的话可以直接看我的源代码配置信息：[围观](https://github.com/ysqi/yushuangqi.com)。