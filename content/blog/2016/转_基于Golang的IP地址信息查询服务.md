
---
date: 2016-12-31T11:32:33+08:00
title: "基于Golang的IP地址信息查询服务"
description: ""
disqus_identifier: 1485833553767375875
slug: "ji-yu-Golangde-IPde-zhi-xin-xi-cha-xun-fu-wu"
source: "https://segmentfault.com/a/1190000008055613"
tags: 
- grpc 
- ip 
- golang 
topics:
- 编程语言与开发
---

原文链接：[http://tabalt.net/blog/ipquer...](http://tabalt.net/blog/ipquery-server-by-golang/)

工作中经常会有通过IP匹配用户信息的需求，如确定用户所在的地区（国家/省份/城市）、运营商、时区、经纬度等等。前一阵有个Golang开发的项目也有这样的需求，于是简单实现了一个包，最近忙里偷闲又包了一个支持HTTP和GRPC方式调用的服务，并开源在GitHub上了。本文主要介绍IP地址信息查询的实现细节和使用方式。

首先交代一下GitHub地址：

-   IpQuery Golang Package：<https://github.com/tabalt/ipquery>

-   IP地址信息查询服务：<https://github.com/tabalt/ipqueryd>

欢迎大家在项目中使用（已通过N亿日PV服务的考验），有任何问题或建议，请提交Issue反馈或Fork到自己名下修改后提交Pull
Request。

### IP数据文件

IP数据文件存放IP地址段和数据信息的映射关系，是IP地址信息查询中最重要的部分，格式上要求可扩展，数据上则需要准确甚至精确。真正意义上完美的IP数据文件是不存在的，而要想让数据文件保持可用，需要定期对数据文件做维护更新。

#### 数据格式

IP数据文件通常是纯文本形式的，映射关系按行存放，每行的各项之间用"t"分隔，前面两列是IP段的起始和结束点转换成无符号32位整型的值（只考虑IPV4），后面的部分则是各项信息，可根据实际需要扩充；各行之间要求是按IP段从小到大升序排列的。示例格式如下：

    1033224192    1033228287    北京    北京    朝阳    联通
    1033228288    1033232383    北京    北京    海淀    联通
    1033232384    1033233407    北京    北京    昌平    联通

#### 数据来源

IP信息数据主要有3个来源：

-   具有一定规模的公司大都会自己维护一份IP库，如果你在这些公司工作，可以直接使用

-   网络上有一些免费的IP库（如纯真IP库）

-   购买商业的IP库（如IPIP.NET）

此外也能通过一定的技术或人工手段自行获取维护IP信息数据库，但是成本会非常高。

### IP地址信息查询的原理

有了数据文件，要实现信息查询并不难，简单方式是直接将数据文件加载到内存数组中，查找时将IP地址转换成无符号32位整型，然后用二分查找法查找整型所在的区间，找到后则返回对应的数据，没找到则返回失败。Golang中核心代码如下：

将IP地址字符串转成无符号32位整型：

    func ip2Long(ip string) uint32 {
        var long uint32
        binary.Read(bytes.NewBuffer(net.ParseIP(ip).To4()), binary.BigEndian, &long)
        return long
    }

主要结构体：

    type IpRange struct {
        Begin uint32
        End   uint32
        Data  []byte
    }

    type IpData []*IpRange

二分查找：

    func (id *IpData) getIpRange(ip string) (*IpRange, error) {
        var low, high int = 0, (id.Length() - 1)

        ipdt := *id
        il := ip2Long(ip)
        if il <= 0 {
            return nil, ErrorIpRangeNotFound
        }

        for low <= high {
            var middle int = (high-low)/2 + low

            ir := ipdt[middle]

            if il >= ir.Begin && il <= ir.End {
                return ir, nil
            } else if il < ir.Begin {
                high = middle - 1
            } else {
                low = middle + 1
            }
        }

        return nil, ErrorIpRangeNotFound
    }

### Golang ipquery包介绍

#### ipquery包的用法

ipquery包（[https://github.com/tabalt/ipq...](https://github.com/tabalt/ipquery/)）用起来很简单，导入包后通过`ipquery.Load()`方法初始化加载IP数据文件，然后就可以使用`ipquery.Find()`方法来查询IP地址对应的信息了。示例代码如下：

    package main

    import (
        "fmt"
        "github.com/tabalt/ipquery"
    )

    func main() {
        df := "testdata/test_10000.data"
        err := ipquery.Load(df)
        if err != nil {
            fmt.Println(err)
        }

        ip := "61.149.208.1"
        dt, err := ipquery.Find(ip)

        if err != nil {
            fmt.Println(err)
        } else {
            fmt.Println(ip, string(dt))
        }
    }

如果你想在程序运行过程中安全地更新数据文件，请使用`ipquery.ReLoad()`方法；`ipquery.Length()`则可以获取到加载到内存的数据总条数。

上面介绍的方法其实都是为了方便使用而包装的快捷方法，也可以直接使用`ipquery.NewIpData()`方法返回的IpData结构体获得更大的灵活性。如给IpData结构体的Load或ReLoad方法传入一个自定义的io.Reader可以从非文本文件的数据源初始化ipquery包。

#### ipquery包的压测结果

ipquery包提供了较完善的单元测试，克隆代码到GOPATH中后，进入\$GOPATH/ipqeury目录，执行`go test`相关命令即可执行测试代码：

    [tabalt@localhost ipquery] go test -v
    === RUN   TestIpData_Load
    --- PASS: TestIpData_Load (0.01s)
    === RUN   TestIpData_Find
    --- PASS: TestIpData_Find (0.01s)
    === RUN   TestIpData_Parallel_Find
    --- PASS: TestIpData_Parallel_Find (0.01s)
    PASS
    ok      ipquery 0.051s

从压测结果上看ipquery包的性能是相当不错的，在一台2核4G CentOS 6.2 Golang
1.7.1虚拟机开发机上，初始化23M的数据文件平均耗时500ms左右，执行查找平均耗时0.012ms，具体数据如下：

    [tabalt@localhost ipquery] go test -bench=.
    BenchmarkIpData_Load-2                 3         452223279 ns/op        97439626 B/op    1780052 allocs/op
    BenchmarkIpData_Find-2            100000             11472 ns/op            1118 B/op         21 allocs/op
    PASS
    ok      ipquery 33.488s
    [tabalt@localhost ipquery] go test -bench=.
    BenchmarkIpData_Load-2                 3         500309108 ns/op        97439621 B/op    1780052 allocs/op
    BenchmarkIpData_Find-2            100000             11809 ns/op            1118 B/op         21 allocs/op
    PASS
    ok      ipquery 33.498s
    [tabalt@localhost ipquery] go test -bench=.
    BenchmarkIpData_Load-2                 3         436756760 ns/op        97439621 B/op    1780052 allocs/op
    BenchmarkIpData_Find-2            100000             12574 ns/op            1118 B/op         21 allocs/op
    PASS
    ok      ipquery 34.510s

### IP地址信息查询服务介绍

如文章开头所说，这个项目基于ipquery包提供HTTP和GRPC接口，名字也就很俗的取为ipqueryd。个人习惯项目级的Go代码不放在全局的GOPATH里，而是使用shell脚本来动态修改GOPATH为项目目录后执行go命令，因此可以使用如下步骤运行本项目：

    [tabalt@localhost ~] git clone https://github.com/tabalt/ipqueryd.git ~/$NOT_YOUR_GOPATH/
    [tabalt@localhost ipqueryd] cd ~/$NOT_YOUR_GOPATH/ipqueryd
    [tabalt@localhost ipqueryd] ./ctrl.sh run

#### 配置文件

项目中conf目录下有个ipqueryd.json的配置文件，可以配置PID文件、HTTP服务端口、GRPC服务端口、数据文件路径等内容，可以根据需求修改；服务端口可以只配其中一个也可以两个都配上。

    {
        "pid_file": "./tmp/ipqueryd.pid",
        "http_server_port": ":12101",
        "grpc_server_port": ":12102",
        "data_file": "./data/ip_data.txt"
    }

#### HTTP接口

HTTP接口支持返回JSON格式和JSONP格式的响应，下面使用命令行测试：

    [tabalt@localhost ipqueryd] curl "http://127.0.0.1:12101/find?ip=1.1.8.1"
    {"data":["广东省电信"]}

    [tabalt@localhost ipqueryd] curl "http://127.0.0.1:12101/find?ip=1.1.8.1&_callback=showip"
    showip({"data":["广东省电信"]});

#### GRPC接口

GRPC接口需要使用以你熟悉的语言编写客户端，下面的代码是Golang中的简单使用：

    package main

    import (
        "log"

        "golang.org/x/net/context"
        "google.golang.org/grpc"

        "pb"
    )

    func main() {
        conn, err := grpc.Dial("127.0.0.1:12102", grpc.WithInsecure())
        if err != nil {
            log.Fatalf("did not connect: %v", err)
        }
        defer conn.Close()

        iqc := pb.NewIpQueryClient(conn)

        ip := "1.1.8.1"

        r, err := iqc.Find(context.Background(), &pb.IpFindRequest{Ip: ip})
        if err != nil {
            log.Fatalf("could not find: %v", err)
        }
        log.Printf("ip data: %s", r.Data)
    }

更多内容等你来发现和贡献！

如果文章对您有帮助，欢迎打赏， 您的支持是我码字的动力！\

原文链接：[http://tabalt.net/blog/ipquer...](http://tabalt.net/blog/ipquery-server-by-golang/)

