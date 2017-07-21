
---
date: 2016-12-31T11:34:44+08:00
title: "Go:gitgofmtpre-commithook"
description: ""
disqus_identifier: 1485833684326072763
slug: "Go:git-gofmt-pre-commit-hook"
source: "https://segmentfault.com/a/1190000000664372"
tags: 
- golang 
topics:
- 编程语言与开发
---

我们使用 git 做 Go
源码的版本控制，在提交前，我们需要对代码自动格式，并且当未格式的时候，输出一些信息，下面是一个这样的脚本：

> 注：该脚本来源于：<http://golang.org/misc/git/pre-commit>\
> 关于介绍清理 Go 代码的小文章：[An Introduction to Go Tools and Clean
> Code](http://www.simontaranto.com/2014/09/06/an-introduction-to-go-tools-and-clean-code.html)

        #!/bin/sh
            # Copyright 2012 The Go Authors. All rights reserved.
            # Use of this source code is governed by a BSD-style
            # license that can be found in the LICENSE file.

            # git gofmt pre-commit hook
            #
            # To use, store as .git/hooks/pre-commit inside your repository and make sure
            # it has execute permissions.
            #
            # This script does not handle file names that contain spaces.

            gofiles=$(git diff --cached --name-only --diff-filter=ACM | grep '.go$')
            [ -z "$gofiles" ] && exit 0

            unformatted=$(gofmt -l $gofiles)
            [ -z "$unformatted" ] && exit 0

            # Some files are not gofmt'd. Print message and fail.

            echo >&2 "Go files must be formatted with gofmt. Please run:"
            for fn in $unformatted; do
                echo >&2 "  gofmt -w $PWD/$fn"
            done

            exit 1

