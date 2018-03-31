
---
date: 2016-12-31T11:33:56+08:00
title: "go-oci8编译安装过程(有别于官方说明)"
description: ""
disqus_identifier: 1485833636916696105
slug: "go-oci8bian-yi-an-zhuang-guo-cheng-(you-bie-yu-guan-fang-shui-ming-)"
source: "https://segmentfault.com/a/1190000004504170"
tags: 
- oracle 
- golang 
categories:
- 编程语言与开发
---

> go-oci8下载地址以及说明地址：<https://github.com/wendal/go-oci>
>
> go-oci8官方说明

1.  安装最新版的git,并设置可以从命令行直接调用git命令

2.  这个步骤多一些

        2.1 下载OCI最新版,存放于C:\instantclient_11_2
        2.2 下载OCI SDK最新版,存放于C:\instantclient_11_2\sdk
        2.3 下载MinGW最新版,安装在C:\mingw
        2.4 下载pkg-config.exe和oci8.pc (已经存放在windows文件夹下)
        2.5 将pkg-config.exe复制到C:\mingw\bin\下
        2.6 将oci8.pc复制到C:\mingw\lib\pkg-config\下

3.  修改系统环境变量,添加

<!-- -->

        PATH=原有PATH;C:\instantclient_11_2;C:\mingw\bin;
        PKG_CONFIG_PATH=C:\mingw\lib\pkg-config

1.  执行 `go get github.com/wendal/go-oci8`

    其中第二步下载，可以在
    [这里](https://github.com/downloads/wendal/go-oci8/mingw_oci_win32.7z)
    下载到，其中的环境已经配置好了\
    如果这一步没在go-oci8官方下载，而是和我当时一样，一步步去各个软件官网下载，那么要注意了

------------------------------------------------------------------------

1.  下载并安装完之后，需要修改oci8.pc；

<!-- -->

        # Package Information for pkg-config
         
        prefix=C:/instantclient_11_2
        exec_prefix=C:/instantclient_11_2
        libdir=${exec_prefix}
        includedir=${prefix}/sdk/include/
         
        Name: OCI
        Description: Oracle database engine
        Version: 11.2
        Libs: -L${libdir} -loci
        Libs.private: 
        Cflags: -I${includedir}

    需要把prefix、exec_prefix指向你的安装目录

1.  这个包是不支持win x64的，我也没测试64位版本的安装

2.  这时候，`go get github.com/wendal/go-oci`

3.  如果和我一样出现了

           C:\Documents and Settings\Administrator>go get github.com/wendal/go-oci8
           # github.com/wendal/go-oci8
           C:\GoPath\src\github.com\wendal\go-oci8\oci8.go:119: cannot use (**C.struct_OCIS erver)(unsafe.Pointer(&conn.svc)) (type **C.struct_OCIServer) as type **C.struct
           _OCISvcCtx in argument to _Cfunc_OCILogon
           C:\GoPath\src\github.com\wendal\go-oci8\oci8.go:136: cannot use (*C.struct_OCIServer)(c.svc) (type *C.struct_OCIServer) as type *C.struct_OCISvcCtx in argument to _Cfunc_OCILogoff
           C:\GoPath\src\github.com\wendal\go-oci8\oci8.go:263: cannot use (*C.struct_OCIServer)(s.c.svc) (type *C.struct_OCIServer) as type *C.struct_OCISvcCtx in argument to _Cfunc_OCIStmtExecute
           C:\GoPath\src\github.com\wendal\go-oci8\oci8.go:383: cannot use (*C.struct_OCIServer)(s.c.svc) (type *C.struct_OCIServer) as type *C.struct_OCISvcCtx in argument to _Cfunc_OCIStmtExecute

> 这样的出错提示，表示你安装的oci版本和go-oci8不一致，这个时候找到`%GOPATH%\src\github.com\wendal\go-oci8\oci8.go`文件

> **有4个地方需要修改**

-   119行

<!-- -->

    (**C.OCIServer)(unsafe.Pointer(&conn.svc)),
    改为
    (**C.OCISvcCtx)(unsafe.Pointer(&conn.svc)),

-   136行

<!-- -->

    (*C.OCIServer)(c.svc),
    改为
    (*C.OCISvcCtx)(c.svc),

-   263行

<!-- -->

    (*C.OCIServer)(c.svc),
    改为
    (*C.OCISvcCtx)(c.svc),

-   383行

<!-- -->

    (*C.OCIServer)(c.svc),
    改为
    (*C.OCISvcCtx)(c.svc),

> 或者直接替换OCIServer为OCISvcCtx。 重新执行
>
>     go get github.com/wendal/go-oci

搞定收工。

