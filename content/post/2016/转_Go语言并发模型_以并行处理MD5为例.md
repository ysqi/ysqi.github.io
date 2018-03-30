
---
date: 2016-12-31T11:33:12+08:00
title: "Go语言并发模型:以并行处理MD5为例"
description: ""
disqus_identifier: 1485833592372590902
slug: "Goyu-yan-bing-fa-mo-xing-:yi-bing-hang-chu-li-MD5wei-li"
source: "https://segmentfault.com/a/1190000006670880"
tags: 
- golang 
topics:
- 编程语言与开发
---

简介
----

Go语言的并发原语允许开发者以类似于 Unix Pipe 的方式构建数据流水线 (data
pipelines)，数据流水线能够高效地利用 I/O和多核 CPU 的优势。

本文要讲的就是一些使用流水线的一些例子，流水线的错误处理也是本文的重点。

阅读建议
--------

本文是["Go语言并发模型：像Unix
Pipe那样使用channel"](https://segmentfault.com/a/1190000006261218)\
一文的下半部分，但重点在于实践。如果你对 channel
已经比较熟悉，则可以独立阅读。\
如果你对 channel 和 go 两个关键字不太熟悉，建议先阅读上半部分。

本文所使用的例子是批量计算文件的MD5值，实现了 linux 下的 md5sum 命令。\
我们首先会讲到 md5sum 的单线程版本，逐步深入到并发的初级和高级版本。

本文中绝大多数讲解都是基于代码进行的。在文章末尾"相关链接"中可以下载三个版本的
md5sum 的实现。

单线程版的 md5sum
-----------------

MD5 是一种广泛用于文件校验的 hash 算法。Linux 下的 md5sum
命令会打印一组文件的 md5值。它的使用方式如下：

    % md5sum *.go
    c33237079343a4d567a2a29df0b8e46e  bounded.go
    a7e3771f2ed58d4b34a73566d93ce63a  parallel.go
    1dc687202696d650594aaac56d579179  serial.go

我们的示例程序类似于
md5sum，但是它接收文件夹作为参数，并打印出每个文件的
md5值，打印结果按照路径排序。\
下面这个例子是 打印当前目录下所有文件的 md5 值：

    % go run serial.go .
    c33237079343a4d567a2a29df0b8e46e  bounded.go
    a7e3771f2ed58d4b34a73566d93ce63a  parallel.go
    1dc687202696d650594aaac56d579179  serial.go

程序的 main 函数调用辅助函数
MD5All，它会返回路径名称到md5值的一个映射。main
函数中对结果进行排序以后，打印出来：

    func main() {
        // 计算特定目录下所有文件的 md5值， 
        // 然后按照路径名顺序打印结果
        m, err := MD5All(os.Args[1])
        if err != nil {
            fmt.Println(err)
            return
        }
        var paths []string
        for path := range m {
            paths = append(paths, path)
        }
        sort.Strings(paths)
        for _, path := range paths {
            fmt.Printf("%x  %s\n", m[path], path)
        }
    }

本文中，函数 MD5All 是讨论的焦点。在
[serial.go](http://oat5ddzns.bkt.clouddn.com/src/piplines/serial.go)的实现中，我们没有使用并发，而是逐个读取和计算
filepath.Walk 生成的目录和文件。代码如下：

    // MD5All 读取 root 目录下的所有文件，返回一个map
    // 该 map 存储了 文件路径到文件内容 md5值的映射
    // 如果 Walk 执行失败，或者 ioutil.ReadFile 读取失败，
    // MD5All 都会返回错误
    func MD5All(root string) (map[string][md5.Size]byte, error) {
        m := make(map[string][md5.Size]byte)
        err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            if !info.Mode().IsRegular() {
                return nil
            }
            data, err := ioutil.ReadFile(path)
            if err != nil {
                return err
            }
            m[path] = md5.Sum(data)
            return nil
        })
        if err != nil {
            return nil, err
        }
        return m, nil
    }

上面的代码中，[filepath.Walk](https://golang.org/pkg/path/filepath/#Walk)
接收两个参数，文件路径和函数指针。\
只要是函数签名和返回值 满足
`func(string, os.FileInfo, error) error`，均可以作为第二参数传递给
filepath.Walk。

点击
[serial.go](http://oat5ddzns.bkt.clouddn.com/src/piplines/serial.go)
下载单线程版本的 md5sum。

并发版的 md5sum
---------------

点击
[parallel.go](http://oat5ddzns.bkt.clouddn.com/src/piplines/parallel.go)
下载并发版 md5sum 的代码。

在这个版本的实现中，我们把 MD5All 切割成两个阶段的流水线。\
第一阶段是 sumFiles，它遍历文件树，每个文件都在一个新的 goroutine
里计算md5值，然后将结果发送到一个result 类型的channel里。\
result 类型的定义如下：

    type result struct {
        path string
        sum  [md5.Size]byte
        err  error
    }

sumFiles 返回两个 channel，一个用于接收 md5计算的结果，一个用于接收
filepath.Walk 产生的错误。\
Walk 函数为每一个文件创建一个 goroutine，然后检查 done channel。如果
done channel 被关闭，walk 函数立即停止执行。代码示例如下：

    func sumFiles(done <-chan struct{}, root string) (<-chan result, <-chan error) {
        // 对于每一个普通文件，启动一个 gorotuine 计算文件 md5 值，
        // 然后 将结果发送到 c。
        // walk 的错误结果发送到 errc。
        c := make(chan result)
        errc := make(chan error, 1)
        go func() {
            var wg sync.WaitGroup
            err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
                if err != nil {
                    return err
                }
                if !info.Mode().IsRegular() {
                    return nil
                }
                wg.Add(1)
                go func() {
                    data, err := ioutil.ReadFile(path)
                    select {
                    case c <- result{path, md5.Sum(data), err}:
                    case <-done:
                    }
                    wg.Done()
                }()
                // done channel 关闭时，终止 walk 函数
                select {
                case <-done:
                    return errors.New("walk canceled")
                default:
                    return nil
                }
            })
            // Walk 函数已经返回，所以 所有对 wg.Add 的调用都会结束
            // 启动一个 goroutine， 它会在所有发送都结束时，关闭 c。
            go func() {
                wg.Wait()
                close(c)
            }()
            // 这里不需要 select 语句，应为 errc 是缓冲管道
            errc <- err
        }()
        return c, errc
    }

MD5All 从 c 接收 md5值。 MD5All 遇到错误时会提前返回，通过 defer
语句关闭 done channel：

    func MD5All(root string) (map[string][md5.Size]byte, error) {
        // MD5All 在函数返回时关闭 done channel
        // 在从 c 和 errc 接收数据前，也可能关闭
        done := make(chan struct{})
        defer close(done)

        c, errc := sumFiles(done, root)

        m := make(map[string][md5.Size]byte)
        for r := range c {
            if r.err != nil {
                return nil, r.err
            }
            m[r.path] = r.sum
        }
        if err := <-errc; err != nil {
            return nil, err
        }
        return m, nil
    }

限制并发量
----------

在 [并发版 MD5All
(parallel.go)](http://oat5ddzns.bkt.clouddn.com/src/piplines/parallel.go)
的实现中，\
我们为每个文件创建了一个
goroutine。如果一个目录中包含很多大文件，可能出现OOM。

我们对并发读取的文件数目稍作限制，进而限制内存的分配。点击
[bounded.go](http://oat5ddzns.bkt.clouddn.com/src/piplines/bounded.go)\
查看限制并发版本的 md5sum。 为了实现限制的目的，我们创建固定数量的
goroutine 用于读取文件。\
这里的流水线包含三个阶段：遍历文件和目录、读取并计算md5值、搜集和整合计算结果。

第一阶段时 walkFiles，它生成一个目录下每个普通文件的路径。代码如下：

    func walkFiles(done <-chan struct{}, root string) (<-chan string, <-chan error) {
        paths := make(chan string)
        errc := make(chan error, 1)
        go func() {
            // Walk 函数返回时，关闭 channel paths
            defer close(paths)
            // 这里不需要select，因为 errc 是缓冲 channel
            errc <- filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
                if err != nil {
                    return err
                }
                if !info.Mode().IsRegular() {
                    return nil
                }
                select {
                case paths <- path:
                case <-done:
                    return errors.New("walk canceled")
                }
                return nil
            })
        }()
        return paths, errc
    }

第二阶段创建固定个数的goroutine digester，每个 digester 从 paths channel
读取文件名，并将结果发送给 c。代码如下：

    func digester(done <-chan struct{}, paths <-chan string, c chan<- result) {
        for path := range paths {
            data, err := ioutil.ReadFile(path)
            select {
            case c <- result{path, md5.Sum(data), err}:
            case <-done:
                return
            }
        }
    }

不像前面的例子，这里 digester 没有关闭输出 channel c，因为 多个 digester
在共享这个channel。\
关闭操作放到 MD5All 中实现，当所有 digester 运行结束时，MD5All
关闭这个channel。代码如下：

        // 启动固定数量的 goroutine 处理文件
        c := make(chan result)
        var wg sync.WaitGroup
        const numDigesters = 20
        wg.Add(numDigesters)
        for i := 0; i < numDigesters; i++ {
            go func() {
                digester(done, paths, c)
                wg.Done()
            }()
        }
        go func() {
            wg.Wait()
            close(c)
        }()

我们可以让每个 digester 创建和返回自己的输出
channel。如果这样做，我们还需要额外的 goroutine 去合并结果。

第三阶段从 channel c 接收结果，并从 channel errc
读取错误信息并执行检查。\
检查操作不能在 c 读取结束之前完成，因为 walkFiles
函数可能会被阻塞而无法向下游阶段发送数据。 代码如下：

    // ... 省略部分代码 ...
        m := make(map[string][md5.Size]byte)
        for r := range c {
            if r.err != nil {
                return nil, r.err
            }
            m[r.path] = r.sum
        }
        // Check whether the Walk failed.
        if err := <-errc; err != nil {
            return nil, err
        }
        return m, nil
    } 

关于Go语言并发模型，使用 Go内置的 channel 类型和 go
关键字实现高并发和并发控制的主题就先到这里。\
在最近发布的 go 1.7中，在核心库中广泛加入了对 context
的支持，以便更好地控制并发和超时。但在这之前\
golang.org/x/net/context 包就一直存在，下一期我们将对 [context
包](https://blog.golang.org/context)及其应用场景进行讨论。

相关链接
--------

1.  [原文链接](https://blog.golang.org/pipelines)

2.  [serial.go](http://oat5ddzns.bkt.clouddn.com/src/piplines/serial.go)

3.  [parallel.go](http://oat5ddzns.bkt.clouddn.com/src/piplines/parallel.go)

4.  [bounded.go](http://oat5ddzns.bkt.clouddn.com/src/piplines/bounded.go)

5.  [golang.org/x/net/context](https://blog.golang.org/context)

扫码关注微信公众号“深入Go语言”



