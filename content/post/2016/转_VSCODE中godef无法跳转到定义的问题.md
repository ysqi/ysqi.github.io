
---
date: 2016-12-31T11:33:18+08:00
title: "VSCODE中godef无法跳转到定义的问题"
description: ""
disqus_identifier: 1485833598515162985
slug: "VSCODEzhong-godefmo-fa-tiao-zhuai-dao-ding-yi-de-wen-ti"
source: "https://segmentfault.com/a/1190000006143167"
tags: 
- visual-studio-code 
- golang 
topics:
- 编程语言与开发
---

原文链接：<http://targetliu.com/vscode-can-not-go-to-def/>

> 之前研究GOLANG时一直用LiteIDE，不得不说，LiteIDE的确不错，但是总感觉缺乏美感，是一款很中规中矩的编辑器。网上看到大家对VSCODE评价不错，尝试后发现的确不错，布局简洁、插件化、支持中文，通过VSCODE
> GO扩展能够很舒服的写GO的代码。

问题描述
--------

不过在实际使用过程中发现 `net` 包无法正常跳转到定义，如下段代码
`ResolveTCPAddr`就无法正常跳转

    package main

    import (
        "net"
    )

    func main() {
        _, err := net.ResolveTCPAddr("tcp", ":4040")
    }

由于VSCODE GO中跳转到定义使用的是godef，遂通过godef的debug模式查看\
问题原因：

    godef -debug -f main.go net.ResolveTCPAddr

运行结果如下：

    2016/08/02 01:17:30 exprType tuple:false pkg: *ast.SelectorExpr net.ListenTCP [
    2016/08/02 01:17:30 exprType tuple:false pkg: *ast.Ident net [
    2016/08/02 01:17:30 exprType tuple:false pkg: *ast.ImportSpec "net" [
    2016/08/02 01:17:30 ] -> 0x0, Type{package "" *ast.ImportSpec "net"}
    2016/08/02 01:17:30 ] -> 0xc820157860, Type{package "" *ast.ImportSpec "net"}
    2016/08/02 01:17:30 member Type{package "" *ast.ImportSpec "net"} 'ListenTCP' {
    2016/08/02 01:17:30     /usr/local/go/src/net/cgo_android.go:10:8: cannot find identifier for package "C": cannot find package "C" in any of:
        /usr/local/go/src/vendor/C (vendor tree)
        /usr/local/go/src/C (from $GOROOT)
        /Users/targetliu/dev/govendor/src/C (from $GOPATH)
        /Users/targetliu/dev/golang/src/C
    2016/08/02 01:17:30 } -> <nil>
    2016/08/02 01:17:30 ] -> 0x0, Type{bad "" <nil> }
    2016/08/02 01:17:30 exprType tuple:false pkg: *ast.SelectorExpr net.ListenTCP [
    2016/08/02 01:17:30 exprType tuple:false pkg: *ast.Ident net [
    2016/08/02 01:17:30 exprType tuple:false pkg: *ast.ImportSpec "net" [
    2016/08/02 01:17:30 ] -> 0x0, Type{package "" *ast.ImportSpec "net"}
    2016/08/02 01:17:30 ] -> 0xc820157860, Type{package "" *ast.ImportSpec "net"}
    2016/08/02 01:17:30 member Type{package "" *ast.ImportSpec "net"} 'ListenTCP' {
    2016/08/02 01:17:30     /usr/local/go/src/net/cgo_android.go:10:8: cannot find identifier for package "C": cannot find package "C" in any of:
        /usr/local/go/src/vendor/C (vendor tree)
        /usr/local/go/src/C (from $GOROOT)
        /Users/targetliu/dev/govendor/src/C (from $GOPATH)
        /Users/targetliu/dev/golang/src/C
    2016/08/02 01:17:30 } -> <nil>
    2016/08/02 01:17:30 ] -> 0x0, Type{bad "" <nil> }
    godef: no declaration found for net.ListenTCP

注意到这一句:

    cannot find identifier for package "C": cannot find package "C" in any of:

原来是 `net` 包里 `import C`
，然而C并不是一个具体真实存在的包，所以godef无法进行分析，导致找不到定义。

godef的GitHub上作者也发现了同样的问题：[Issue:net.LookupIP fails
\#41](https://github.com/rogpeppe/godef/issues/41)

解决方案
--------

在godef的GitHub上看到有人提交了针对这个问题的解决方案：[master - Special
treatment for "C" package.
\#44](https://github.com/rogpeppe/godef/pull/44)

根据这个提交，可以尝试使用如下方法解决：

-   找到并打开godef的 `go/parser/parser.go` 这个文件

-   在 `1970行` 左右添加(代码中+号部分，可以通过搜索定位)：

        if declIdent == nil {
                      filename := p.fset.Position(path.Pos()).Filename
                      name, err := p.pathToName(litToString(path), filepath.Dir(filename))
         +            if litToString(path) == "C" {
         +                name = "C"
         +            }
                      if name == "" {
                          p.error(path.Pos(), fmt.Sprintf("cannot find identifier for package %q: %v", litToString(path), err))
                      } else {

-   重新编译godef

如果遇到同样问题的同学不妨试一试以上方式，至少对于我来说，问题得到了解决。\
也希望作者能尽快修复这个问题。

