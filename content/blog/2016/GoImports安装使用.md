---
date: 2016-02-25T13:32:49+08:00
description: ""
disqus_identifier: "20160001"
slug: "gosublime-goimports"
tags:
- Go
- Tool
- Sublime
title: GoImports安装使用
topics:
- 开发
---

Go开发过程中时候总需要手工引入包和删除没有使用的包，此时有人意识到可以改善下，想Java、C#、Python等等，总能自动的帮你处理下包。现在Go官方提供了 GoImports 工具，而在 Sublime Text 下也提供了相关的插件来支持自动检测。

这里介绍的是由 [Brad Fitz](https://github.com/bradfitz) 提供的 GoImport 插件，该插件基于 GoImprots 工具在 Sublime Text 的插件。 非常好用，能在保存 Go 文件时自动帮你格式化文件并检查包的引用使用，如果使用的包没有导入，则自动导入。相反，当导入的包没有被使用时，则自动删除。

### 如何配置安装 GoImports 工具

1. 确保路径`$GOPATH/bin`在环境变量`$PATH`中。Windows对应：`%GOPATH%\bin`在环境变量`%PATH%`中。

2. 运行命令获取 GoImprots 工具包
```bash
go get golang.org/x/tools/cmd/goimports
``` 
运行后，能直接运行`$ goimprots`命令，已检查工具是否安装成功。

3. 在 Sublime Text 中安装插件 [GoSublime](https://github.com/DisposaBoy/GoSublime)

4. 安装后打开 GoSublime 的配置文件

5. 在配置文件中添加新的根配置项：
```
{
    "fmt_cmd" :[ "goimports"]
}
```

### 总结

现在，你就可以又开心又方便的使用 Sublime Text 编辑 Go 文件，当你保持文件时，都能够自动添加/删除引入包了，再也没有烦人的编译错误。