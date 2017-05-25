
---
date: 2017-05-24T09:17:30+08:00
title: "golang使用原始套接字构造UDP包"
description: ""
disqus_identifier: 1495588650161107373
slug: "golangshi-yong-yuan-shi-tao-jie-zi-gou-zao-UDPbao"
source: "https://segmentfault.com/a/1190000009253715"
tags: 
- linux 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

RAW SOCKET 介绍
---------------

TCP/IP协议中，最常见的就是原始(SOCKET\_RAW)、tcp(SOCKET\_STREAM)、udp(SOCKET\_DGRA)三种套接字。原始套接字能够对底层传输进行控制，允许自行组装数据包，比如修改本地IP，发送Ping包，进行网络监听。这里不做详细介绍，要了解更多可以网上自己查询。

实现
----

这里先看IP头结构：

其中16位总长度包括IP头长度和数据的长度，8位协议填写17，因为UDP协议类型为17。这里要说明一下IP头中的首部校验，这个值只校验IP头部，不包含数据。\
这里给出校验算法，IP头和UDP头中使用的校验算法是一样的。

    func checkSum(msg []byte) uint16 {
        sum := 0
        for n := 1; n < len(msg)-1; n += 2 {
            sum += int(msg[n])*256 + int(msg[n+1])
        }
        sum = (sum >> 16) + (sum & 0xffff)
        sum += (sum >> 16)
        var ans = uint16(^sum)
        return ans
    }

下面开始填充IP头，这里使用了golang.org/x/net下的ipv4包

        //目的IP
        dst := net.IPv4(192, 168, 1, 2)
        //源IP
        src := net.IPv4(192, 168, 1, 3)
        //填充ip首部
        iph := &ipv4.Header{
            Version:  ipv4.Version,
            //IP头长一般是20
            Len:      ipv4.HeaderLen,
            TOS:      0x00,
            //buff为数据
            TotalLen: ipv4.HeaderLen + len(buff),
            TTL:      64,
            Flags:    ipv4.DontFragment,
            FragOff:  0,
            Protocol: 17,
            Checksum: 0,
            Src:      src,
            Dst:      dst,
        }
        
        h, err := iph.Marshal()
        if err != nil {
            log.Fatalln(err)
        }
        //计算IP头部校验值
        iph.Checksum = int(checkSum(h))

下面开始处理UDP头部，先来看UDP头结构：

UDP头结构就很简单了，16位UDP校验和涉及到一个UDP伪首部的东西，我们先来看下UDP伪首部的构成。

    -----------------------------------------
    |         32bit Source IP address       |
    -----------------------------------------
    |         32bit Destination IP addr     |
    -----------------------------------------
    |  0   | 8bit Proto| 16bit header length|
    -----------------------------------------

伪首部包含了源IP，目的IP，协议号，16位的长度。这个伪首部仅仅参与校验计算。\
下面开始填充UDP头：

        //填充udp首部
        //udp伪首部
        udph := make([]byte, 20)
        //源ip地址
        udph[0], udph[1], udph[2], udph[3] = src[12], src[13], src[14], src[15]
        //目的ip地址
        udph[4], udph[5], udph[6], udph[7] = dst.IP[12], dst.IP[13], dst.IP[14], dst.IP[15]
        //协议类型
        udph[8], udph[9] = 0x00, 0x11
        //udp头长度
        udph[10], udph[11] = 0x00, byte(len(buff)+8)
        //下面开始就真正的udp头部
        //源端口号
        udph[12], udph[13] = 0x27, 0x10
        //目的端口号
        udph[14], udph[15] = 0x17, 0x70
        //udp头长度
        udph[16], udph[17] = 0x00, byte(len(buff)+8)
        //校验和
        udph[18], udph[19] = 0x00, 0x00
        //计算校验值
        check := checkSum(append(udph, buff...))
        udph[18], udph[19] = byte(check>>8&255), byte(check&255）

下面我们需要发送自己构造的UDP包，可以使用net下的ListenPacket。

        listener, err := net.ListenPacket("ip4:udp", "192.168.1.104")
        if err != nil {
            log.Fatal(err)
        }
        defer listener.Close()
        
        //listener 实现了net.PacketConn接口
        r, err := ipv4.NewRawConn(c)
        if err != nil {
            log.Fatal(err)
        }

        //发送自己构造的UDP包
        if err = r.WriteTo(iph, append(udph[12:20], buff...), nil); err != nil {
            log.Fatal(err)
        }

这个实现只在linux和mac上测试过，windows上需要借助于第三方吧，比如winpcap。

结语
----

这里只给出了UDP的实现，TCP的实现比较复杂，以后也会给出TCP实现的例子。

