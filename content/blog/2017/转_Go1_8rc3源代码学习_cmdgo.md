
---
date: 2017-02-17T08:17:13+08:00
title: "Go1_8rc3源代码学习:cmdgo"
description: ""
disqus_identifier: 1487290633588074516
slug: "Go-1_8rc3-yuan-dai-ma-xue-xi-:cmd-go"
source: "https://segmentfault.com/a/1190000008318284"
tags: 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

前言
====

命令行工具 go 相关的代码在 \<go-src\>/src/cmd/go，目录结构

    <go-src>/src/cmd/go
        internal
        testdata
        alldocs.go
        go11.go
        go_test.go
        go_unix_test.go
        go_windows_test.go
        main.go
        mkalldocs.sh
        note_test.sh
        note_test.go
        vendor_test.go

-   main.go，入口函数

-   \*\_test.go，单元测试代码

-   internal，go 内部实现相关代码

internal 目录下基本上按照 go "子命令" 进行组织，可以看到常用的子命令比如
help, list, run .etc

    <go-src>/src/cmd/go
        internal
            base
            bug
            get
            help
            list
            run
            ...

main.go
=======

main.go 是 go 命令的入口，为了优雅的支持各种 "子命令"，main.go
将各个子命令对象保存在数组里，通过遍历数组找到具体的子命令，然后调用各个子命令的
run 方法

    for _, cmd := range base.Commands {
        if cmd.Name() == args[0] && cmd.Runnable() {
            cmd.Flag.Usage = func() { cmd.Usage() }
            if cmd.CustomFlags {
                args = args[1:]
            } else {
                cmd.Flag.Parse(args[1:])
                args = cmd.Flag.Args()
            }
            cmd.Run(cmd, args)
            base.Exit()
            return
        }
    }

Command struct
--------------

A command is an implementation of a go command

    type Command struct {
        // Run runs the command.
        // The args are the arguments after the command name.
        Run func(cmd *Command, args []string)

        // UsageLine is the one-line usage message.
        // The first word in the line is taken to be the command name.
        UsageLine string

        // Short is the short description shown in the 'go help' output.
        Short string

        // Long is the long message shown in the 'go help <this-command>' output.
        Long string

        // Flag is a set of flags specific to this command.
        Flag flag.FlagSet

        // CustomFlags indicates that the command will do its own
        // flag parsing.
        CustomFlags bool
    }

go 支持的所有 Command 存放在 base package 的 Command 数组 Command，并在
main.go 的 init 函数中初始化

    func init() {
            base.Commands = []*base.Command{
            work.CmdBuild,
            clean.CmdClean,
            doc.CmdDoc,
            envcmd.CmdEnv,
            bug.CmdBug,
            fix.CmdFix,
            fmtcmd.CmdFmt,
            generate.CmdGenerate,
            get.CmdGet,
            work.CmdInstall,
            list.CmdList,
            run.CmdRun,
            test.CmdTest,
            tool.CmdTool,
            version.CmdVersion,
            vet.CmdVet,
            .etc
        }
    }

go build
========

build command 的相关代码在
\<go-src\>/src/cmd/go/internal/work/build.go，在讨论具体的流程之前，我们先来看一些和
build 相关的数据结构

toolchain interface
-------------------

toolchain 接口定义了在编译过程中一些通用的方法

    type toolchain interface {
        // gc runs the compiler in a specific directory on a set of files
        // and returns the name of the generated output file.
        gc(b *Builder, p *load.Package, archive, obj string, asmhdr bool, importArgs []string, gofiles []string)   
    (ofile string, out []byte, err error)
        // cc runs the toolchain's C compiler in a directory on a C file
        // to produce an output file.
        cc(b *Builder, p *load.Package, objdir, ofile, cfile string) error
        // asm runs the assembler in a specific directory on specific files
        // and returns a list of named output files.
        asm(b *Builder, p *load.Package, obj string, sfiles []string) ([]string, error)
        // pkgpath builds an appropriate path for a temporary package file.
        Pkgpath(basedir string, p *load.Package) string
        // pack runs the archive packer in a specific directory to create
        // an archive from a set of object files.
        // typically it is run in the object directory.
        pack(b *Builder, p *load.Package, objDir, afile string, ofiles []string) error
        // ld runs the linker to create an executable starting at mainpkg.
        ld(b *Builder, root *Action, out string, allactions []*Action, mainpkg string, ofiles []string) error
        // ldShared runs the linker to create a shared library containing the pkgs built by toplevelactions
        ldShared(b *Builder, toplevelactions []*Action, out string, allactions []*Action) error

        compiler() string
        linker() string
    }

### noToolchain

### goToolchain

### gccgoToolchain

Action
------

An action represents a single action in the action graph \
Action 结构封装了 build 过程中的一个阶段性的"动作"，Deps
字段描述它依赖哪些 Action，triggers 字段描述哪些 Action
依赖它，通过这两个字段，所有的 Action 组成一个 action graphic

    type Action struct {
        ...
        Deps []*Action
        triggers []*Action
        ...
    }

Builder
-------

A Builder holds global state about a build. It does not hold per-package
state, because we build packages in parallel, and the builder is shared.

    type Builder struct {
        ...
        ready     actionQueue
    }

总结
====

