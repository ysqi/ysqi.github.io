
---
date: 2016-12-31T11:33:37+08:00
title: "在CentOS下利用FFMPEG对视频进行转码"
description: ""
disqus_identifier: 1485833617045630571
slug: "zai-CentOSxia-li-yong-FFMPEGdui-shi-pin-jin-hang-zhuai-ma"
source: "https://segmentfault.com/a/1190000005121784"
tags: 
- centos 
- ffmpeg 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

先按照ffmpeg的安装攻略，搞定你CentOS上的ffmpeg，我目前使用的版本是3.0.2\
然后直接上代码

    package main

    import (
        "fmt"
        "log"
        "os"
        "os/exec"
        "time"
    )

    func main() {
        var srcFileName, outputfilename string
        log.SetFlags(log.Lshortfile | log.Ldate | log.Ltime)

        if len(os.Args) != 3 {
            fmt.Println("简单的转码服务，利用FFMPEG和编码器转换成MP4.")
            fmt.Println("使用：[输入文件路径] [输出文件路径]")
            os.Exit(1)
        } else {
            srcFileName = os.Args[1]
            outputfilename = os.Args[2]
        }
        param := "ffmpeg -y -i " + srcFileName + " -metadata:s:v rotate=0  -vf fps=15,setdar=dar=1 -s 480x480 -pix_fmt yuv420p -strict -2 -c:v h264 -b:v 500k -b:a 48k -ss 0 -t 300 -threads 2 " + outputfilename + ""
        fmt.Println("parm : ", param)
        start := time.Now()
        _, err := exec.Command("bash", "-c", param).CombinedOutput()
        if err != nil {

            fmt.Println("error: " + err.Error())

        } else {
            fmt.Println(" out file : ", outputfilename, " exec time  ", time.Now().Sub(start).Seconds())
        }
    }

执行下：

    go run main.go simple.mov simple_1.mp4

