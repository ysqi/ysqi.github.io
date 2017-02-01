
---
date: 2016-12-31T11:33:18+08:00
title: "Golang模仿七牛图片处理API"
description: ""
disqus_identifier: 1485833598994592068
slug: "Golangmo-fang-qi-niu-tu-pian-chu-li-API"
source: "https://segmentfault.com/a/1190000006088096"
tags: 
- golang 
topics:
- 编程语言与开发
---

> 之前一直在用qiniu的存储服务，生成图片的缩略图，模糊图，视频的webp，现在需要把存储移到s3上，那么这些图片，视频处理就要自己动手写了，本文梳理一下大致的思路。

分析需求
--------

先看一下qiniu的接口是如何处理图片的，例如先截取视频第一秒的图片，再把图片缩略，最后存储到一个新的key，命令可以这么写
`vframe/jpg/offset/1|imageMogr2/thumbnail/400x|saveas/xxx`,
可以看到三个操作之间用 `|` 符号分割，类似unix 的 pipe 操作。

上面的操作算作一个`cmd`,
一次API请求可以同时处理多个`cmd`，`cmd`之间用分号分割,
处理完毕后，在回调中把处理结果返回，例如

    {
        "id": "xxxxx", 
        "pipeline": "xxx", 
        "code": 0, 
        "desc": "The fop was completed successfully", 
        "reqid": "xTsAAFnxUbR5J10U", 
        "inputBucket": "xxx", 
        "inputKey": "xxxxx", 
        "items": [
            {
                "cmd": "vframe/jpg/offset/1|imageMogr2/thumbnail/400x|saveas/ZmFtZS1wcml2YXRlOm1vbWVudC9jb3Zlci9zbmFwL3ZpZGVvL2M5YzdjZjQ5LTU3NGQtNGZjMS1iZDFkLTRkYjZkMzlkZWY1Ni8wLzA=", 
                "code": 0, 
                "desc": "The fop was completed successfully", 
                "hash": "FhdN6V8EI4vW4XJGALSfxutvMEIv", 
                "key": "xx", 
                "returnOld": 0
            }, 
            {
                "cmd": "vframe/jpg/offset/1|imageMogr2/thumbnail/400x|imageMogr2/blur/45x8|saveas/ZmFtZS1wcml2YXRlOm1vbWVudC9jb3Zlci9zbmFwL3ZpZGVvL2M5YzdjZjQ5LTU3NGQtNGZjMS1iZDFkLTRkYjZkMzlkZWY1Ni8wLzBfYmx1cg==", 
                "code": 0, 
                "desc": "The fop was completed successfully", 
                "hash": "FgNiRzrCsa7TZx1xVSb_4d5TiaK3", 
                "key": "xxx", 
                "returnOld": 0
            }
        ]
    }

分解需求
--------

这个程序大致需要这么几个部分:

1.  一个http接口，接受任务，接受后把任务扔到队列，返回一个job ID。
    worker异步处理任务，worker的个数 和 每个worker 并行的处理的个数
    能够配置，worker有重试机制。

2.  从 job payload 中解析出需要做的任务，解析出每个cmd,
    最好能并行执行每一个 cmd, 记录每一个cmd的结果

3.  每个cmd中有多个 `operation`, 并且用 pipe
    连接，前一个operaion的输出是后一个operation的输入

可以把 1 和 2，3 分开来看，1
比较独立，之前写过一个worker的模型，参考的是这篇文章 [Handling 1 Million
Requests per Minute with
Go](http://marcio.io/2015/07/handling-1-million-requests-per-minute-with-golang/)，比较详细，是用
go channel 作为queue的，我加了一个 beanstalk 作为 queue的
providor。还有一点改进是，文章中只提供了worker数量的设置，我再加了一个参数，设定每个worker可以并行执行的协程数。所以下面主要讲讲3，
2的解决办法

### Pipe

可以参考这个库 [pipe](http://gopkg.in/pipe.v2), 用法如下：

    p := pipe.Line(
        pipe.ReadFile("test.png"),
        resize(300, 300),
        blur(0.5),
    )

    output, err := pipe.CombinedOutput(p)
    if err != nil {
        fmt.Printf("%v\n", err)
    }

    buf := bytes.NewBuffer(output)
    img, _ := imaging.Decode(buf)

    imaging.Save(img, "test_a.png")

还是比较方便的，建一个 `Cmd` struct, 利用正则匹配一下每个 `Operation`
的参数，放入一个 `[]Op` slice, 最后执行，struct和方法如下：

    type Cmd struct {
        cmd    string
        saveas string
        ops    []Op
        err    error
    }

    type Op interface {
        getPipe() pipe.Pipe
    }

    type ResizeOp struct {
        width, height int
    }

    func (c ResizeOp) getPipe() pipe.Pipe {
        return resize(c.width, c.height)
    }

    //使用方法
    cmdStr := `file/test.png|thumbnail/x300|blur/20x8`
    cmd := Cmd{cmdStr, "test_b.png", nil, nil}

    cmd.parse()
    cmd.doOps()

### sync.WaitGroup

单个cmd处理解决后，就是多个cmd的并行问题，没啥好想的，直接用
`sync.WaitGroup`
就可以完美解决。一步一步来，我们先看看这个struct的使用方法：

    func main() {
        cmds := []string{}
        for i := 0; i < 10000; i++ {
            cmds = append(cmds, fmt.Sprintf("cmd-%d", i))
        }

        results := handleCmds(cmds)

        fmt.Println(len(results)) // 10000
    }

    func doCmd(cmd string) string {
        return fmt.Sprintf("cmd=%s", cmd)
    }

    func handleCmds(cmds []string) (results []string) {
        fmt.Println(len(cmds)) //10000
        var count uint64

        group := sync.WaitGroup{}
        lock := sync.Mutex{}
        for _, item := range cmds {
            // 计数加一
            group.Add(1)
            go func(cmd string) {
                result := doCmd(cmd)
                atomic.AddUint64(&count, 1)

                lock.Lock()
                results = append(results, result)
                lock.Unlock()
                
                // 计数减一
                group.Done()
            }(item)
        }

        // 阻塞
        group.Wait()

        fmt.Printf("count=%d \n", count) // 10000
        return
    }

group本质大概是一个计数器，计数 &gt; 0时, `group.Wait()` 会阻塞，直到
计数 == 0. 这里还有一点要注意，就是 `results = append(results, result)`
的操作是线程不安全的，清楚这里 results
是共享的，需要加锁来保证同步，否则最后 `len(results)` 不为 10000。

我们建一个`BenchCmd`， 来存放 cmds. 如下：

    type BenchCmd struct {
        cmds      []Cmd
        waitGroup sync.WaitGroup
        errs      []error
        lock      sync.Mutex
    }

    func (b *BenchCmd) doCmds() {
        for _, item := range b.cmds {
            b.waitGroup.Add(1)

            go func(cmd Cmd) {
                cmd.parse()
                err := cmd.doOps()

                b.lock.Lock()
                b.errs = append(b.errs, err)
                b.lock.Unlock()

                b.waitGroup.Done()
            }(item)
        }

        b.waitGroup.Wait()
    }

最后的调用就像这样：

    var cmds []Cmd
    cmd_a := Cmd{`file/test.png|thumbnail/x300|blur/20x8`, "test_a.png", nil, nil}
    cmd_b := Cmd{`file/test.png|thumbnail/500x1000|blur/20x108`, "test_b.png", nil, nil}
    cmd_c := Cmd{`file/test.png|thumbnail/300x300`, "test_c.png", nil, nil}

    cmds = append(cmds, cmd_a)
    cmds = append(cmds, cmd_b)
    cmds = append(cmds, cmd_c)

    bench := BenchCmd{
        cmds:      cmds,
        waitGroup: sync.WaitGroup{},
        lock:      sync.Mutex{},
    }

    bench.doCmds()

    fmt.Println(bench.errs)

这只是一个初级的实验，思考还不够全面，并且只是模仿API，qiniu应该不是这么做的，耦合更低，可能各个Cmd都有各自处理的集群，那`pipe`这个库就暂时没法解决了，目前的局限在于
每个Cmd必须都在一个进程中。

