
---
date: 2016-12-31T11:32:42+08:00
title: "mockgo程序的新方法"
description: ""
disqus_identifier: 1485833562880503968
slug: "mock-go-cheng-xu-de-xin-fang-fa"
source: "https://segmentfault.com/a/1190000007733142"
tags: 
- 单元测试 
- mock 
- golang 
categories:
- 编程语言与开发
---

一直以来，我都认为在 go 里面 mock 是非常困难的。不像动态语言或者跑在 VM
上的语言，go 要求在开发的时候就给 mock
介入预留空间，不然测试的时候会不得其门而入。开发的时候需要头疼的事情可多了，还要求再考虑下可测试性，真有点强人所难。另外第三方库并不一定给
mock 预留空间，遇到这种情况只能干瞪眼绕路走。很多时候，无法 mock
掉某些带副作用的函数，就不能覆盖掉目标路径。既然测试不到关键的路径，那干脆就不写测试了。结果是，项目里很多
go 代码事实上一直都没有被测试覆盖掉。

但最近我发现了一个库：<https://github.com/bouk/monkey>\
似乎可以跟开头的烦恼永别了？小范围地体验了下，感觉还是挺好用的。

长话短说，`monkey`
库通过修改内存地址的方式，替换目标函数的实际执行地址，实现（几乎）任意函数的
mock。你可以指定目标函数，然后定义一个匿名函数替换掉它。替换的记录会存在一个全局表里，不需要的时候可以通过它重新恢复原来的目标函数。由于采用的是修改内存地址的黑科技，作者建议千万不要用在测试环境以外的地方。目前仅支持x86架构上的
Linux 和 Mac，Windows 似乎没有测试过？不管怎样，支持 Linux 和 Mac
就足以覆盖开发机和 CI 环境了。

`monkey` 库用起来非常简单，直接边上示例代码，边解释好了：

    package main

    import (
        "fmt"
        "github.com/bouk/monkey"
        "os"
        "os/exec"
        "reflect"
        "testing"
    )

    // 假如我们要测试函数 call
    func call(cmd string) (int, string) {
        bytes, err := exec.Command("sh", "-c", cmd).CombinedOutput()
        output := string(bytes)
        if err != nil {
            return 1, reportExecFailed(output)
        }
        return 0, output
    }

    // 上面的函数会调用它，这个函数一定要mock掉！
    func reportExecFailed(msg string) string {
        os.Exit(1) // 讨人嫌的副作用
        return msg
    }

    func TestExecSussess(t *testing.T) {
        // 恢复 patch 修改
        // 实际使用中会把 UnpatchAll 放到 teardown 函数里
        // 不过在 go 自带的 testing 里就这么处理了
        defer monkey.UnpatchAll()
        // mock 掉 exec.Command 返回的 *exec.Cmd 的 CombinedOutput 方法
        monkey.PatchInstanceMethod(
            reflect.TypeOf((*exec.Cmd)(nil)),
            "CombinedOutput", func(_ *exec.Cmd) ([]byte, error) {
                return []byte("results"), nil
            },
        )
        // mock 掉 reportExecFailed 函数
        monkey.Patch(reportExecFailed, func(msg string) string {
            return msg
        })

        rc, output := call("any")
        if rc != 0 {
            t.Fail()
        }
        if output != "results" {
            t.Fail()
        }
    }

    func TestExecFailed(t *testing.T) {
        defer monkey.UnpatchAll()
        // 上次 mock 的是执行成功的情况，这一次轮到执行失败
        monkey.PatchInstanceMethod(
            reflect.TypeOf((*exec.Cmd)(nil)),
            "CombinedOutput", func(_ *exec.Cmd) ([]byte, error) {
                return []byte(""), fmt.Errorf("sth bad happened")
            },
        )
        monkey.Patch(reportExecFailed, func(msg string) string {
            return msg
        })

        rc, output := call("any")
        if rc != 1 {
            t.Fail()
        }
        if output != "" {
            t.Fail()
        }
    }

执行 `go test xx_test.go`，可以运行上面的代码。

测试中常有的一个需求：在位置 A 需要 mock 掉函数，在位置 B
里需要调用原来的函数才能运行下去。这时候需要使用 `monkey` 库提供的
`PatchGuard` 结构体。官方文档中有个示例，这里稍微调整下：

    package main

    import (
        "fmt"
        "github.com/bouk/monkey"
        "strings"
    )

    func main() {
        var guard *monkey.PatchGuard
        guard = monkey.Patch(fmt.Println, func(a ...interface{}) (n int, err error) {
            s := make([]interface{}, len(a))
            for i, v := range a {
                s[i] = strings.Replace(fmt.Sprint(v), "hell", "*bleep*", -1)
            }
            // 以下代码等价于
            // guard.Unpatch()
            // defer guard.Restore()
            // return fmt.Println(s...)
            guard.Unpatch()
            n, err = fmt.Println(s...)
            guard.Restore()
            return
        })
        fmt.Println("what the hell?") // what the *bleep*?
        fmt.Println("what the hell?") // what the *bleep*?
    }

上面的代码关键在于，调用原来的函数之前先调用一次 `Unpatch`，恢复到 mock
之前的情况；然后在调用了原函数之后，调用一次 `Restore`，重新打上
mock。剩下的，无非是根据输入参数来判断现在是运行到位置 A，还是位置 B。

