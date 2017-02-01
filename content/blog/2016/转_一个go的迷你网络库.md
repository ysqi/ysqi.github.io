
---
date: 2016-12-31T11:34:26+08:00
title: "一个go的迷你网络库"
description: ""
disqus_identifier: 1485833666329813002
slug: "yi-ge-gode-mi-ni-wang-lao-ku"
source: "https://segmentfault.com/a/1190000002982372"
tags: 
- golang 
topics:
- 编程语言与开发
---

go语言完善的基础设施为编写网络程序提供了极大的便利.只需要少量代码就可以编写一个高性能,稳定的异步网络程序.\
本文介绍一个迷你的,基于事件回调的异步网络库.

首先简单介绍一下并发模型.

go提供了基于goroutine的同步网络接口,所以对每个网络连接可以创建一个单独的goroutine用于接收网络数据.这个goroutine是执行一个死循环,不断的recv数据,解包然后将完整的逻辑包发送到一个每连接唯一的chan中,供逻辑消费.

除了网络接收goroutine之外,每个连接还有一个专门的处理逻辑消息的goroutine,它的工作就是不断的从关联的chan中提取逻辑包并处理.

与之前的chuck-lua类似,为了让使用者可以方便定制自己的包结构,我提供了packet和decoder的抽象.

packet.go

    package packet

    const (
        RAWPACKET = 1
        RPACKET   = 2
        WPACKET   = 3
        EPACKET   = 4
    )

    type Packet interface{
        MakeWrite()(*Packet)
        MakeRead() (*Packet)
        Clone()    (*Packet)
        PkLen()    (uint32)
        DataLen()  (uint32)
        Buffer()   (*ByteBuffer)
        GetType()  (byte)
    }

decoder.go

    package packet

    import(
        "net"
        "encoding/binary"
        "fmt"
        "io"
    )

    var (
        ErrPacketTooLarge     = fmt.Errorf("Packet too Large")
        ErrEOF                = fmt.Errorf("Eof")
    )

    type Decoder interface{
        DoRecv(Conn net.Conn)(Packet,error)
    }

    type RPacketDecoder struct{
        maxpacket uint32
    }

    func NewRPacketDecoder(maxpacket uint32)(RPacketDecoder){
        return RPacketDecoder{maxpacket:maxpacket}
    }

    func (this RPacketDecoder)DoRecv(Conn net.Conn)(Packet,error){
        header := make([]byte,4)
        n, err := io.ReadFull(Conn, header)
        if n == 0 && err == io.EOF {
            return nil,ErrEOF
        }else if err != nil {
            return nil,err
        }
        size := binary.LittleEndian.Uint32(header)
        if size > this.maxpacket {
            return nil,ErrPacketTooLarge
        }
        buf := make([]byte,size+4)
        copy(buf[:],header[:])
        n, err = io.ReadFull(Conn,buf[4:])
        if n == 0 && err == io.EOF {
            return nil,ErrEOF
        }else if err != nil {
            return nil,err
        }
        return NewRPacket(NewBufferByBytes(buf,(uint32)(len(buf)))),nil
    }

    type RawDecoder struct{
    }

    func NewRawDecoder()(RawDecoder){
        return RawDecoder{}
    }

    func (this RawDecoder)DoRecv(Conn net.Conn)(Packet,error){
        buff  := make([]byte,4096)
        n,err := Conn.Read(buff)
        if n == 0 && err == io.EOF {
            return nil,ErrEOF
        }else if err != nil {
            return nil,err
        }
        return NewRawPacket(NewBufferByBytes(buff,(uint32)(n))),nil     
    }

内置了2种包的类型,分别是rawpacket,rpacket/wpacket.

rawpacket其实就是原始二进制数据流,没有区分逻辑界限.

rpacket/wpacket则提供了一种4字节包头,的二进制流包结构.

eventpacket则作为内部使用,目前用来向逻辑通告连接关闭,错误等事件.

下面就是整个网络库的核心部分tcpsession,它提供了对tcp连接的处理.

    package tcpsession

    import(
           "net"
           packet "kendynet-go/packet"
           "fmt"
       )

    var (
        ErrUnPackError     = fmt.Errorf("TcpSession: UnpackError")
        ErrSendClose       = fmt.Errorf("send close")
        ErrSocketClose     = fmt.Errorf("socket close")
    )


    type Tcpsession struct{
        Conn         net.Conn
        Packet_que   chan packet.Packet
        decoder      packet.Decoder
        socket_close bool
        ud           interface{}
    }

    func (this *Tcpsession) SetUd(ud interface{}){
        this.ud = ud
    }

    func (this *Tcpsession) Ud()(interface{}){
        return this.ud
    }

    func dorecv(session *Tcpsession){
        for{
            p,err := session.decoder.DoRecv(session.Conn)
            if session.socket_close{
                break
            }
            if err != nil {
                session.Packet_que <- packet.NewEventPacket(err)
                break
            }
            session.Packet_que <- p 
        }
        close(session.Packet_que)
    }


    func ProcessSession(tcpsession *Tcpsession,decoder packet.Decoder,
                        process_packet func (*Tcpsession,packet.Packet,error))(error){
        if tcpsession.socket_close{
            return ErrSocketClose
        }
        tcpsession.decoder = decoder
        go dorecv(tcpsession)
        for{
            msg,ok := <- tcpsession.Packet_que
            if !ok {
                //log error
                return nil
            }
            if packet.EPACKET == msg.GetType(){
                process_packet(tcpsession,nil,msg.(packet.EventPacket).GetError())
            }else{
                process_packet(tcpsession,msg,nil)
            }
            if tcpsession.socket_close{
                return nil
            }
        }
    }

    func NewTcpSession(conn net.Conn)(*Tcpsession){
        session := new(Tcpsession)
        session.Conn = conn
        session.Packet_que   = make(chan packet.Packet,1024)
        session.socket_close = false
        return session
    }

    func (this *Tcpsession)Send(wpk packet.Packet)(error){
        if this.socket_close{
            return ErrSocketClose
        }
        idx := (uint32)(0)
        for{
            buff  := wpk.Buffer().Bytes()
            end   := wpk.PkLen()
            n,err := this.Conn.Write(buff[idx:end])
            if err != nil || n < 0 {
                return ErrSendClose
            }
            idx += (uint32)(n)
            if idx >= (uint32)(end){
                break
            }
        }
        return nil
    }

    func (this *Tcpsession)Close(){
        if this.socket_close{
            return
        }
        this.socket_close = true
        this.Conn.Close()
    }

代码十分简短只有一百行出头点,这里关键地方时,dorecv,Send和ProcessSession三个函数.

dorecv所做的就是不断调用decoder.DoRecv从网络中提取网络包,然后将其写入到Packet\_que中.

Send函数则保证需要发送的数据被写入到内湖缓冲或出错才会返回.

ProcessSession则是整个库的核心所在,接收到新连接之后,用新建连接作为参数调用ProcessSession,当网络包到达或出错时将会回调使用者提供的回调函数.从实现看它只是简单的创建一个goroutine执行dorecv,然后在一个for循环中不断读取到达的网络包然后调用回调函数.

下面是一个使用示例,更多的示例请参考:<https://github.com/sniperHW/kendynet-go>

    package main

    import(
        "net"
        tcpsession "kendynet-go/tcpsession"
        packet "kendynet-go/packet"
        "fmt"
    )

    func main(){
        service := ":8010"
        tcpAddr,err := net.ResolveTCPAddr("tcp4", service)
        if err != nil{
            fmt.Printf("ResolveTCPAddr")
        }
        listener, err := net.ListenTCP("tcp", tcpAddr)
        if err != nil{
            fmt.Printf("ListenTCP")
        }
        for {
            conn, err := listener.Accept()
            if err != nil {
                continue
            }
            session := tcpsession.NewTcpSession(conn)
            fmt.Printf("a client comming\n")
            go tcpsession.ProcessSession(session,packet.NewRawDecoder(),
               func (session *tcpsession.Tcpsession,rpk packet.Packet,errno error){ 
                if rpk == nil{
                    fmt.Printf("%s\n",errno)
                    session.Close()
                    return
                }
                session.Send(rpk)
               })
        }
    }

