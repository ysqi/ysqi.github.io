
---
date: 2017-02-18T11:13:41+08:00
title: "Go最新的dep详解"
description: ""
disqus_identifier: 1487387621585643189
slug: "Gozui-xin-de-depxiang-jie"
source: "https://mp.weixin.qq.com/s?__biz=MjM5OTcxMzE0MQ==&amp;mid=2653369860&amp;idx=1&amp;sn=0e3b9ab8346e69087bb2050b28b34d9f&amp;chksm=bce4d61e8b935f0817f34510225ababb71491b5a5614c2b6ec105a3f103347fbb7cccabf57e8&amp;mpshare=1&amp;scene=1&amp;srcid=0214qm8bmviC4MFY2xKkygJG&amp;key=81e78a55b343"
tags: 
- golang 
topics:
- 编程语言与开发
---

该文翻译自https://medium.com/i-can-haz-downtime/dep-101-c85e8ab6ed45\#.hbngswi0e

我很高兴在过去几个月和几个其他gopher开发的一个原型依赖管理工具，名为dep。

dep是去年开始由Peter
Bourgon组织的项目的一部分。由于我参与开发了一个“godep”，Go的OG依赖管理工具（继承自Keith
Rarick），因此加入了dep项目的团队。

除了我自己和Peter，团队的其他成员是Jessie Frazelle，Andrew Gerrand和Sam
Boyer。 Andrew是Google Go team的一员。
Jessie在Google工作，并参与过大型Go项目，如Docker和Kubernetes。
Sam维护gps。

该团队发布了一系列我们工作过程中的进展信息。到目前为止，各种其他工具作者和相关方也以不同的方式参与这一项目。

起初 {style="margin: 1.3em 0px 1em; padding: 0px; font-weight: bold;font-size: 1.4em; border-bottom: 1px solid rgb(238, 238, 238);"}
----

假设我们正在使用github.com/gorilla/mux编写一个Web应用程序。
这里是一些代码，让我们开始：

``` {style="font-size: 0.85em; font-family: Consolas, Inconsolata, Courier, monospace;font-size: 1em; line-height: 1.2em;margin: 1.2em 0px;"}
package main

import (
    "net/http"
    "os"

    "github.com/gorilla/mux"
)

func main() {
    r := mux.NewRouter()
    r.Handle("/", http.FileServer(http.Dir(".")))
    http.ListenAndServe(":"+os.Getenv("PORT"), r)
}
```

在现有项目上第一次使用dep时，需要运行dep init。

dep
init将在GOPATH中已经包含github.com/gorilla/mux，manifest.json文件将包括它。我先运行如下命令：

``` {style="font-size: 0.85em; font-family: Consolas, Inconsolata, Courier, monospace;font-size: 1em; line-height: 1.2em;margin: 1.2em 0px;"}
go get -u github.com/gorilla/mux
```

所以在我的\$GOPATH中github.com/gorilla/mux的分支是master。如果我的\$GOPATH中的版本匹配Semver
tag（例如：v1.2.3），那么将使用该tag的名称。

dep可以跨越架构和go版本。我们可以将github.com/gorilla/mux与旧版本的Go（\<1.7.0）的github.com/gorilla/context包结合使用。当我最后运行

``` {style="font-size: 0.85em; font-family: Consolas, Inconsolata, Courier, monospace;font-size: 1em; line-height: 1.2em;margin: 1.2em 0px;"}
go get -u github.com/gorilla/mux
```

的时候，我运行的是Go
1.7.5，所以github.com/gorilla/context包不在我的\$GOPATH。因为可能是编译项目所需的依赖，它会包含在lock.json文件中。在这种场景下，如果从属项目有一个semver兼容的release
tag，dep会选择最新的版本。在这种情况下是github.com/gorilla/context的v1.1。

因为github.com/gorilla/mux不包含manifest.json文件，dep不知道github.com/gorilla/mux目前是否能与github.com/gorilla/context@v1.1配合使用。dep一般使用在依赖的manifest.json文件中找到的约束。

dep
init包括所有依赖关系（包括递归依赖）以及在lock.json文件中使用的确切版本。

对于示例应用程序，这将创建以下两个文件：\
lock.json

``` {style="font-size: 0.85em; font-family: Consolas, Inconsolata, Courier, monospace;font-size: 1em; line-height: 1.2em;margin: 1.2em 0px;"}
{
    "memo": "d741a3bed21fe6cae9d67c523b0a343859882b2f246f2a293e2676cfacd5a2ce",
    "projects": [
        {
            "name": "github.com/gorilla/context",
            "version": "v1.1",
            "revision": "a85d2e53ba63bdea074dbbbb5983f0516974e87b",
            "packages": [
                "."
            ]
        },
        {
            "name": "github.com/gorilla/mux",
            "branch": "master",
            "revision": "392c28fe23e1c45ddba891b0320b3b5df220beea",
            "packages": [
                "."
            ]
        }
    ]
}
```

manifest.json

``` {style="font-size: 0.85em; font-family: Consolas, Inconsolata, Courier, monospace;font-size: 1em; line-height: 1.2em;margin: 1.2em 0px;"}
{
    "dependencies": {
        "github.com/gorilla/mux": {
            "branch": "master"
        }
    }
}
```

确保项目可以编译 {style="margin: 1.3em 0px 1em; padding: 0px; font-weight: bold;font-size: 1.4em; border-bottom: 1px solid rgb(238, 238, 238);"}
----------------

在运行dep init之后，应该运行dep
ensure以便将构建项目所需的软件包副本填充在vendor/目录。
这确保任何项目的依赖项都包括在lock文件和vendor目录中。
如果你想确保记录所有的依赖，运行dep ensure。

添加依赖 {style="margin: 1.3em 0px 1em; padding: 0px; font-weight: bold;font-size: 1.4em; border-bottom: 1px solid rgb(238, 238, 238);"}
--------

添加另一个依赖项，只需在代码中使用它。
当需要检查你的工作时，你需要运行dep ensure更新lock.json文件和vendor/。
这将锁定项目到每个依赖项的最新发布版本。

如果最新版本不合适呢？ {style="margin: 1.3em 0px 1em; padding: 0px; font-weight: bold;font-size: 1.4em; border-bottom: 1px solid rgb(238, 238, 238);"}
----------------------

如果需要指定版本，可以使用ensure命令的备用形式，如：dep ensure
github.com/gorilla/mux@\\\^1.3.0。该命令修改manifest.json文件，约束dep使用github.com/gorilla/mux的\\\^v1.3.0版本，解析依赖关系树并更新vendor/中的依赖关系。在示例应用程序上，因为dep已经选择了最新的可用版本1.3.0。但是，以后的更新（dep
ensure
-update）将不再跟踪主分支，而是使用semver标记的版本≥1.3.0和\<2.0.0。

dep主要使用Rust
cargo类似的操作符来选择依赖的版本。这些包括\\\^，～和=。还有一个更严格的，向前兼容的匹配比\\\^使用～。例如，dep确保github.com/com/gorilla/mux@\~1.2.0将匹配大于等于1.2.0和小于1.3.0任何版本。要锁定到特定版本，请使用=（e.x.
dep ensure
github.com/com/gorilla/mux@=1.2.0）。在未来，dep计划将\\\^作为默认前缀。

保持更新 {style="margin: 1.3em 0px 1em; padding: 0px; font-weight: bold;font-size: 1.4em; border-bottom: 1px solid rgb(238, 238, 238);"}
--------

要保持项目的依赖关系最新使用dep ensure
-update，它将所有依赖关系更新为由manifest.json中的约束所允许的最新版本，并忽略lock.json的内容。
将新版本写入vendor/，并在lock.json中更新相应的元数据。\
在将来，可以用dep ensure -update 更新单个依赖关系。

Status {style="margin: 1.3em 0px 1em; padding: 0px; font-weight: bold;font-size: 1.4em; border-bottom: 1px solid rgb(238, 238, 238);"}
------

当缺少依赖项时，dep status会告诉您哪些软件包缺失。 例如，这里是dep
status的输出：

``` {style="font-size: 0.85em; font-family: Consolas, Inconsolata, Courier, monospace;font-size: 1em; line-height: 1.2em;margin: 1.2em 0px;"}
$ dep status
PROJECT                 MISSING PACKAGES
github.com/boltdb/bolt  [github.com/boltdb/bolt]
```

如果项目的lock.json是最新的，dep
status命令显示所有依赖项的列表，作为项目，并为每个：

-   应用约束

-   选版本

-   所选修订版本

-   最新版本或修订版本

-   使用的软件包数量。\
    将bolt添加到示例项目并运行dep后，确保更新lock.json和vendor/，dep
    status如下所示：

``` {style="font-size: 0.85em; font-family: Consolas, Inconsolata, Courier, monospace;font-size: 1em; line-height: 1.2em;margin: 1.2em 0px;"}
$ dep status
PROJECT                     CONSTRAINT  VERSION        REVISION  LATEST   PKGS USED
github.com/boltdb/bolt      *           v1.3.0         583e893   v1.3.0   1
github.com/gorilla/context  *           v1.1           a85d2e5   v1.1     1
github.com/gorilla/mux      ^1.3.0      v1.3.0         392c28f   v1.3.0   1
golang.org/x/sys            *           branch master  7a6e564   7a6e564  1
```

这两种模式可以在将来合并。

移除依赖 {style="margin: 1.3em 0px 1em; padding: 0px; font-weight: bold;font-size: 1.4em; border-bottom: 1px solid rgb(238, 238, 238);"}
--------

dep remove删除manifest.json，lock.json和vendor/中的依赖关系。
使用示例应用程序，此提交从应用程序中删除github.com/gorilla/mux的使用，下一个提交是dep
remove github.com/gorilla/mux的结果。
因为github.com/gorilla/mux被删除，github.com/gorilla/context不再需要，也会被删除。
如果在运行dep remove时仍然使用依赖关系，则命令将失败。
可以使用-force强制从manifest.json中删除约束。
但是，由于依赖项仍在使用中，它仍然在lock.json中并复制到vendor/。

未来 {style="margin: 1.3em 0px 1em; padding: 0px; font-weight: bold;font-size: 1.4em; border-bottom: 1px solid rgb(238, 238, 238);"}
----

dep仍然处于实验阶段，有许多问题需要解决和很多工作要做。
我希望这篇文章清楚地显示了一些如何使用工具的例子。
使用方式可能会随着时间的推移通过社区反馈和请求而改变。
如果你有一些空闲时间，并有兴趣参与一些Go工具开发，请现在开始。\
谢谢阅读！

