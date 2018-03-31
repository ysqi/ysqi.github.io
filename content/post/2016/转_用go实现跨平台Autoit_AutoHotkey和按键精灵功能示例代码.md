
---
date: 2016-12-31T11:32:50+08:00
title: "用go实现跨平台Autoit_AutoHotkey和按键精灵功能示例代码"
description: ""
disqus_identifier: 1485833570047059145
slug: "yong-goshi-xian-kua-ping-tai-Autoit_AutoHotkeyhe-an-jian-jing-ling-gong-neng-shi-li-dai-ma"
source: "https://segmentfault.com/a/1190000007590542"
tags: 
- autohotkey 
- golang 
- c 
categories:
- 编程语言与开发
---

Robotgo
v0.40.0对鼠标键盘操作进行了大幅度的完善,由于跨平台切换测试麻烦(合并几个平台的代码到对比上传代码都浪费了不少时间)和C代码生产效率与Go和JavaScript不在一个级别上,没有支持窗口和进程对应窗口的选择以及窗口Bounds,将在下一个版本完善,键盘操作可能在某些输入法上有些问题请切换到系统输入法(目前优先完成窗口句柄的问题,后面会解决该问题).

项目地址:[https://github.com/go-vgo/rob...](https://github.com/go-vgo/robotgo)

下面整理本机测试部分的代码,将会在以后版本陆续完善,并附上注释(请看清代码再运行):

    package main

    import (
        . "fmt"
        "github.com/go-vgo/robotgo"
    )

    func main() {
        //键盘控制
        robotgo.TypeString("Hello World")//输入Hello World
        robotgo.KeyTap("enter")//按下enter键
        robotgo.KeyTap("a", "control")
        robotgo.KeyTap("h", "command") //隐藏窗口

        robotgo.KeyTap("i", "alt", "command")
        //按下"i", "alt", "command"组合键
        arr := []string{"alt", "command"}
        robotgo.KeyTap("i", arr)
        //按下"i", "alt", "command"组合键

        robotgo.KeyTap("w", "command") //关闭窗口
        robotgo.KeyTap("m", "command") //最小化窗口
        robotgo.KeyTap("f1", "control")
        robotgo.KeyTap("a", "control")
        robotgo.KeyToggle("a", "down")//切换a键
        robotgo.KeyToggle("a", "down", "alt")
        robotgo.KeyToggle("a", "down", "alt", "command")
        robotgo.KeyToggle("enter", "down")
        robotgo.TypeString("en")

        //鼠标控制
        robotgo.MoveMouse(100, 200)//移动鼠标到100, 200位置
        robotgo.MouseClick()//鼠标左键单击
        robotgo.MouseClick("right", false) //右键单击
        robotgo.MouseClick("left", true)   //左键双击
        robotgo.ScrollMouse(10, "up")//向上滚动鼠标
        robotgo.MouseToggle("down", "right")//鼠标右键切换
        robotgo.MoveMouseSmooth(100, 200)//平滑移动鼠标到100, 200
        robotgo.MoveMouseSmooth(100, 200, 1.0, 100.0)//设置平滑移动速度
        x, y := robotgo.GetMousePos()//获取鼠标坐标位置
        Println("pos:", x, y)
        if x == 456 && y == 586 {
            Println("mouse...", "586")
        }

        robotgo.MouseToggle("up")
        robotgo.MoveMouse(x, y)
        robotgo.MoveMouse(100, 200)

        for i := 0; i < 1080; i += 1000 {
            Println(i)
            robotgo.MoveMouse(800, i)
        }
        //屏幕控制
        //robotgo.CaptureScreen()
        // bit_map := robotgo.CaptureScreen()
        // Println("CaptureScreen...", bit_map)
        //gbit_map := robotgo.Capture_Screen()
        gbit_map := robotgo.BCaptureScreen()//获取屏幕位图
        Println("Capture_Screen...", gbit_map.Width)

        sx, sy := robotgo.GetScreenSize()//获取屏幕width和height
        Println("...", sx, sy)

        color := robotgo.GetPixelColor(100, 200)//获取坐标100, 200的颜色
        Println("color----", color, "-----------------")

        color2 := robotgo.GetPixelColor(10, 20)//获取坐标10, 20的颜色
        Println("color---", color2)

        // Bitmap
        abit_map := robotgo.CaptureScreen()//获取全屏位图
        Println("a...", abit_map)

        bit_map := robotgo.CaptureScreen(100, 200, 30, 40)
        //获取100, 200, 30, 40的位图
        Println("CaptureScreen...", bit_map)
        // Println("...", bit_map.Width, bit_map.BytesPerPixel)

        fx, fy := robotgo.FindBitmap(bit_map)//查找位图
        Println("FindBitmap------", fx, fy)

        bit_pos := robotgo.GetPortion(bit_map, 10, 10, 11, 10)//截取位图
        Println(bit_pos)

        bit_str := robotgo.TostringBitmap(bit_map)//Tostring位图
        Println("bit_str...", bit_str)

        // sbit_map := robotgo.BitmapFromstring(bit_str, 2)
        // Println("...", sbit_map)

        robotgo.SaveBitmap(bit_map, "test.png")//保存位图为图片
        robotgo.SaveBitmap(bit_map, "test31.tif", 1)
        robotgo.Convert("test.png", "test.tif")//转换位图图片格式

        open_bit := robotgo.OpenBitmap("test.tif")//打开图片位图
        Println("open...", open_bit)

        //全局监听事件
        Println("---请按v键---")
        eve := robotgo.AddEvent("v")

        if eve == 0 {
            Println("---你按下v键---", "v")
        }

        Println("---请按k键---")
        keve := robotgo.AddEvent("k")
        if keve == 0 {
            Println("---你按下k键---", "k")
        }

        Println("---请按鼠标左键---")
        mleft := robotgo.AddEvent("mleft")
        if mleft == 0 {
            Println("---你按下左键---", "mleft")
        }

        // mright := robotgo.AddEvent("mright")
        // if mright == 0 {
        //  Println("---你按下右键---", "mright")
        // }

        // robotgo.LStop()

        //窗口
        abool := robotgo.ShowAlert("hello", "robotgo")//弹出窗口
        if abool == 0 {
            Println("ok@@@", "确认")
        }
        robotgo.ShowAlert("hello", "robotgo", "确认", "取消")
        // robotgo.GetPID()
        mdata := robotgo.GetActive()//获取当前窗口
        hwnd := robotgo.GetHandle()//获取当前窗口hwnd
        Println("hwnd---", hwnd)
        title := robotgo.GetTitle()//获取当前窗口标题
        Println("title-----", title)
        robotgo.CloseWindow()//关闭当前窗口
        robotgo.SetActive(mdata)//SetActive窗口
    }

