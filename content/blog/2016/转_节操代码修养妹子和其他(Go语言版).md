
---
date: 2016-12-31T11:33:14+08:00
title: "节操代码修养妹子和其他(Go语言版)"
description: ""
disqus_identifier: 1485833594425288593
slug: "jie-cao-dai-ma-xiu-yang-mei-zi-he-ji-ta-(Goyu-yan-ban-)"
source: "https://segmentfault.com/a/1190000006233292"
tags: 
- golang 
topics:
- 编程语言与开发
---

Festival & Fuck, Coding, Inner depth, Sister and Others.

某些文章会提到《为什么Go语言这么不受待见》，《真的没必要浪费心思在 Go
语言上》，《我为什么放弃Go语言》，《Why worse is
better》等话题。经常重温这些话题，每次都会有新发现。最忌手里有了一个语言，心里便容不下另一个语言。

忽略细节、语法或者设计，Go语言各种好用。考虑到这些因素，Go被喷出翔都不为过。

本文不打算在细节、语法或者设计上扯淡，只举些例子，说一说如何用Go语言写出还凑合的代码。

### 类、对象、属性，可能还夹杂着一点设计模式

    //代码来自 https://github.com/xgdapg/xconn/blob/master/xconn.go，已验证

    //Conn 对应一个tcp连接
    type Conn struct {
            //原生TCP连接
        conn         net.Conn
            //发送数据的channel（类似队列）
        send         chan *MsgData
            //消息处理方法
        msgHandler   MsgHandler
            //tcp缓冲区
        recvBuffer   []byte
            //消息一次封装，后续会进行二次封装
        msgPacker    MsgPacker
            //健康检查周期
        pingInterval uint
            //健康检查方法
        pingHandler  PingHandler
            //停止健康检查channel
        pingStop     chan bool
            //自定义关闭连接时所调用方法
        closeHandler CloseHandler
    }

    //MsgHandler 为自定义处理消息
    type MsgHandler interface {
        HandleMsg(*MsgData)
    }

    //MsgPacker 为自定义拆包解包方式，为了性能可以直接将此段与recvLoop端合并
    type MsgPacker interface {
        PackMsg(*MsgData) []byte
        UnpackMsg([]byte) *MsgData
    }

    //MsgData 自定义消息类型
    type MsgData struct {
        Data []byte
        Ext  interface{}
    }

    //PingHandler 为自定义心跳命令
    type PingHandler interface {
        HandlePing()
    }

    //CloseHandler 为连接断开时的自定义操作
    type CloseHandler interface {
        HandleClose()
    }

    //NewConn 为处理新连接方式：启动两个协程，一个只负责读，一个只负责写，
    //也可以认为开启了三个协程，第三个协程负责进行定时ping操作
    func NewConn(conn net.Conn) *Conn {
        c := &Conn{
            conn:         conn,
            send:         make(chan *MsgData, 64),
            msgHandler:   nil,
            recvBuffer:   []byte{},
            msgPacker:    nil,
            pingInterval: 0,
            pingHandler:  nil,
            pingStop:     nil,
            closeHandler: nil,
        }

        go c.recvLoop()
        go c.sendLoop()

        return c
    }

    //recoverPanic 程序panic情况下的处理方法（例如向已经关闭的tcp连接写数据会造成panic）
    func recoverPanic() {
        if err := recover(); err != nil {
            //fmt.Println(err)
        }
    }

    //SetMsgHandler 为 Getter Setter 方法
    func (this *Conn) SetMsgHandler(hdlr MsgHandler) {
        this.msgHandler = hdlr
    }

    //SetMsgPacker 为 Getter Setter 方法
    func (this *Conn) SetMsgPacker(packer MsgPacker) {
        this.msgPacker = packer
    }

    //SetPing 为 Getter Setter 方法
    func (this *Conn) SetPing(sec uint, hdlr PingHandler) {
        this.pingInterval = sec
        this.pingHandler = hdlr
        if this.pingStop == nil {
            this.pingStop = make(chan bool)
        }
        if sec > 0 {
            go this.pingLoop()
        }
    }

    //SetCloseHandler 为 Getter Setter 方法
    func (this *Conn) SetCloseHandler(hdlr CloseHandler) {
        this.closeHandler = hdlr
    }

    //pingLoop 为定时健康检查操作
    func (this *Conn) pingLoop() {
        defer recoverPanic()
        for {
            select {
            case <-this.pingStop:
                return
            case <-time.After(time.Duration(this.pingInterval) * time.Second):
                this.Ping()
            }
        }
    }

    //RawConn 返回原始的tcp连接
    func (this *Conn) RawConn() net.Conn {
        return this.conn
    }

    //recvLoop 用于处理接收到的tcp包，并进行拆包等操作，然后调用recvMsg方法进行处理
    func (this *Conn) recvLoop() {
        defer recoverPanic()
        defer this.Close()
        buffer := make([]byte, 2048)
            //一次封包协议：四个字节（int32）表示包长度，根据包长度截取消息长度作为包。
        for {
            bytesRead, err := this.conn.Read(buffer)
            if err != nil {
                return
            }

            this.recvBuffer = append(this.recvBuffer, buffer[0:bytesRead]...)
            for len(this.recvBuffer) > 4 {
                length := binary.BigEndian.Uint32(this.recvBuffer[0:4])
                readToPtr := length + 4
                if uint32(len(this.recvBuffer)) < readToPtr {
                    break
                }
                if length == 0 {
                    if this.pingHandler != nil {
                        this.pingHandler.HandlePing()
                    }
                } else {
                    buf := this.recvBuffer[4:readToPtr]
                    go this.recvMsg(buf)
                }
                this.recvBuffer = this.recvBuffer[readToPtr:]
            }
        }
    }

    //recvMsg 为代理，实际执行的是后台的HandleMsg方法。
    func (this *Conn) recvMsg(data []byte) {
        defer recoverPanic()
        msg := &MsgData{
            Data: data,
            Ext:  nil,
        }
            //调用UnackMsg对信息进行二次解包
        if this.msgPacker != nil {
            msg = this.msgPacker.UnpackMsg(data)
        }
        if this.msgHandler != nil {
            this.msgHandler.HandleMsg(msg)
        }
    }

    //sendLoop 用于发送数据包
    func (this *Conn) sendLoop() {
        defer recoverPanic()
        for {
            msg, ok := <-this.send
            if !ok {
                break
            }

            go this.sendMsg(msg)
        }
    }

    //sendMsg 用于发送数据包，实际先调用PackMsg进行信息持久化，然后二次封包，转换为本框架能接受的形式
    func (this *Conn) sendMsg(msg *MsgData) {
        defer recoverPanic()
        sendBytes := make([]byte, 4)
        if msg != nil {
            data := msg.Data
            if this.msgPacker != nil {
                data = this.msgPacker.PackMsg(msg)
            }
            length := len(data)
            binary.BigEndian.PutUint32(sendBytes, uint32(length))
            sendBytes = append(sendBytes, data...)
        }
        this.conn.Write(sendBytes)
    }

    //Close 关闭连接
    func (this *Conn) Close() {
        defer recoverPanic()
        this.conn.Close()
        close(this.send)
        if this.pingStop != nil {
            close(this.pingStop)
        }
        if this.closeHandler != nil {
            this.closeHandler.HandleClose()
        }
    }

    //SendMsg 用于发送数据
    func (this *Conn) SendMsg(msg *MsgData) {
        this.send <- msg
    }

    //SendData 用于发送数据
    func (this *Conn) SendData(data []byte) {
        this.SendMsg(&MsgData{Data: data, Ext: nil})
    }

    //Ping 用于健康监测
    func (this *Conn) Ping() {
        go this.sendMsg(nil)
    }

作为一个专用于处理TCP链接的框架，实际上xconn（上文中的代码）进行了两次封装，连消息发送、信息拆包封包、甚至接收信息都进行了二次封装。

实际代码中，可以进行简化操作，将二次的部分简化为一次。

将代码写得和上面一样工整，便已经超越大部分猿了。

上个例子中作者用到了recover，用的很克制，却又恰到好处。

### 至于defer和panic

Go语言的try catch?

    import (
        "fmt"
        "github.com/manucorporat/try"
    )

    func main() {
        try.This(func() {
            panic("my panic")

        }).Finally(func() {
            fmt.Println("this must be printed after the catch")

        }).Catch(func(e try.E) {
            // Print crash
            fmt.Println(e)
        })
    }

以上代码纯属搞笑，个人不建议工程项目中使用如此写法，但是这种做法可以借鉴。

工程代码：（用于Go与数据库Transaction）

    //代码来自《Go语言游戏项目应用情况汇报》

    func (db *Database) Transaction(work func()) {
        db.lock.Lock()
        defer db.lock.UnLock()
        //事务控制
        defer func() {
            if err := recover; err == nil {
                db.commit(info)
            } else {
                db.rollback()
                //选择性抛出panic
                panic(TransError{err})
            }
        }()
        //执行传入的函数
        work()
    }

《Go语言游戏项目应用情况汇报》是我所能找到的，为数不多的几个敢开放部分工程代码的分享。整体代码比较整洁，适于新手学习。

以上对err的处理方法写法，和《Errors are values》有异曲同工之妙。

### err的一种处理方式

示范代码：来自《Errors are vales》

    //这种写法强烈不推荐！！！！这就是许多人说的Go程序一大半都在check error
    _, err = fd.Write(p0[a:b])
    if err != nil {
        return err
    }
    _, err = fd.Write(p1[c:d])
    if err != nil {
        return err
    }
    _, err = fd.Write(p2[e:f])
    if err != nil {
        return err
    }
    // and so on
    //重要的事情说两遍：不推荐，但是个人小项目这样写，完全没问题。

推荐如下写法：

    var err error
    write := func(buf []byte) {
        if err != nil {
            return
        }
        _, err = w.Write(buf)
    }
    write(p0[a:b])
    write(p1[c:d])
    write(p2[e:f])
    // and so on
    if err != nil {
        return err
    }

也推荐如下写法：

    func (ew *errWriter) write(buf []byte) {
        if ew.err != nil {
            return
        }
        _, ew.err = ew.w.Write(buf)
    }

其他：

正名：为什么选择Go语言。

答：因为简单，并且也不会别的。

建议：尽量选择Go1.6及以上版本，避免GC造成程序STW。

至于GC性能，可以参考 [Go1.6中的gc
pause已经完全超越JVM了吗?](https://www.zhihu.com/question/42353634)。

原文禁止转载，因此提炼出关键字：从效果看XXX，但XXX；并且，XXX，XXX。

So, Why worse is better?

