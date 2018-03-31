
---
date: 2016-12-31T11:34:52+08:00
title: "Go的sync_mutex实现"
description: ""
disqus_identifier: 1485833692727913803
slug: "Gode-sync_mutexshi-xian"
source: "https://segmentfault.com/a/1190000000506960"
tags: 
- golang 
categories:
- 编程语言与开发
---

概述
----

sync/mutex是Go语言底层基础对象之一，用于构建多个goroutine间的同步逻辑，因此被大量高层对象所使用。\
其工作模型类似于Linux内核的futex对象，具体实现极为简洁，性能也有保证。

数据结构
--------

    type Mutex struct {
        state int32    
        sema  uint32   
    }                  

mutex对象仅有两个数值字段，分为为state（存储状态）和sema（用于计算休眠goroutine数量的信号量）。\
初始化时填入的0值将mutex设定在未锁定状态，同时保证时间开销最小。\
这一特性允许将mutex作为其它对象的子对象使用。

### state字段

state被定义为int32类型，允许为其调用原子方法（sync/atomic），从而原子化地设定状态。\
每个state字段均划分三个状态段，含义如下：

        31          3 2 1 0
        +----~~~----+-+-+-+
        |     S     | |W|L|
        +----~~~----+-+-+-+
         |           | | |
         |           | | \--- 锁定状态，0表示未锁定，1表示已锁定
         |           | |
         |           | \----- 唤醒事件，0表示无事件，1表示mutex已被解除锁定，可以唤醒等待其它goroutine
         |           |  
         |           \------- 保留位，保持为0
         |
         \------------------- 等待唤醒以尝试锁定的goroutine的计数，0表示没有等待者

方法
----

### Lock() / 锁定

    func (m *Mutex) Lock() {
        // 快速路径：直接锁定mutex
        // 记为FG同步点：Fast Grab
        if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
            // 仅在没有等待者、没有唤醒事件、没有锁定的情况下执行
            if raceenabled {
                raceAcquire(unsafe.Pointer(m))
            }
            return
        }

        awoke := false
        for {
            old := m.state
            new := old | mutexLocked
            if old&mutexLocked != 0 {
                // 处于锁定状态，增加等待者计数
                new = old + 1<<mutexWaiterShift
            }
            if awoke {
                // goroutine已被唤醒，“消费”唤醒事件，重置标志位
                new &^= mutexWoken
            }
            // 记为G同步点：Grab
            if atomic.CompareAndSwapInt32(&m.state, old, new) {
                // 新旧状态一致，没有被其它goroutine修改
                if old&mutexLocked == 0 {
                    // 成功锁定
                    break
                }
                // 休眠等待
                runtime_Semacquire(&m.sema)
                awoke = true
            }
            // 新旧状态不一致，重新取状态并尝试锁定
        }

        if raceenabled {
            raceAcquire(unsafe.Pointer(m))
        }
    }

### Unlock() / 解除锁定

    func (m *Mutex) Unlock() {
        if raceenabled {
            _ = m.state
            raceRelease(unsafe.Pointer(m))
        }

        // 快速路径：直接解除锁定
        // 记为FD同步点：Fast Drop
        new := atomic.AddInt32(&m.state, -mutexLocked)
        if (new+mutexLocked)&mutexLocked == 0 {
            // 连续解除两次以上
            panic("sync: unlock of unlocked mutex")
        }

        old := new
        for {
            // 如果没有等待者，或已经有等待者被唤醒，或已经有goroutine锁定mutex，
            // 则无需尝试唤醒
            if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken) != 0 {
                return
            }
            // 产生唤醒事件，尝试唤醒某个等待者
            new = (old - 1<<mutexWaiterShift) | mutexWoken
            // 记为W同步点：Wake Up
            if atomic.CompareAndSwapInt32(&m.state, old, new) {
                // 新旧状态一致，没有被其它goroutine修改
                // 尝试唤醒
                runtime_Semrelease(&m.sema)
                return
            }
            // 新旧状态不一致，重新取状态并尝试唤醒
            old = m.state
        }
    }

可以观察到，两个函数一共有四个原子函数调用点，也就是状态同步点。\
一旦原子函数调用失败，说明状态已被其它goroutine改变，需要重新获取状态以决定后续动作。

### 调用者

mutex不与具体goroutine挂钩，可被任意的goroutine调用，但需要注意一下几点：

1.  同一个goroutine不能在“连续”调用Lock()函数两次，否则会导致死锁；
2.  在确保调用顺序正确的前提下，完全可以在一个goroutine中调用Lock()，在另一个中调用Unlock()；
3.  同一个mutex不能被“连续”调用Unlock()函数两次，否则会导致异常。

状态迁移
--------

### 状态列表

state的三个状态段一共有如下状态组合（`-`表示0）：

  状态组合   休眠者   唤醒事件   锁定
  ---------- -------- ---------- ------
  `---`      -        -          -
  `--L`      -        -          是
  `-WL`      -        有         是
  `S-L`      有       -          是
  `SW-`      有       有         -
  `S--`      有       -          -
  `-W-`      -        有         -
  `SWL`      有       有         是

结合同步点分析，可以得到如下规则：\
1. 非唤醒goroutine在尝试锁定时不消耗唤醒事件，对该状态段不做改变；\
2. 已唤醒goroutine一定会消耗唤醒事件，将重置该状态段；\
3. 解除锁定后才尝试发送唤醒事件。

### 迁移路径

#### 只有一个goroutine调用的情况

这种情况下，状态只在`---`与`--L`之间迁移（路径1和2），且只涉及到FG和D两个同步点。

#### 有两个goroutine调用的情况

这种情况下，状态涉及`---`、`--L`、`S-L`、`S--`、`-W-`、`-WL`六个状态。\
假设两个goroutine分别称为X和Y：

    1. 当X已经锁定mutex，而Y尝试锁定时，会从`--L`迁移到`S-L`（路径3）；  
    2. 当X解除锁定后，会从`S-L`迁移到`S--`（路径5），此时Y还在休眠中；  
    3. 当X发送唤醒事件后，会从`S--`状态迁移到`-W-`（路径7），并尝试唤醒休眠者（即Y），此时有三条路径：  
        3.1 Y成功锁定mutex，则从`-W-`迁移到`--L`（路径8），X若尝试锁定，则进一步迁移到`S-L`（路径3）；
        3.2 X又尝试锁定mutex，则从`-W-`迁移到`-WL`（路径14），Y因被唤醒而尝试锁定，则进一步迁移到`S-L`（路径17）；  
        3.3 X又尝试锁定mutex，且在Y尝试锁定前解除锁定，则从`-W-`迁移到`-WL`（路径14）而后又迁移回`-W-`（路径15）。   

#### 有多个goroutine调用的情况

这种情况是两个goroutine调用情况的延续，在原基础上再涉及`-WL`和`SWL`两个状态。\
假设三个goroutine分别称为X、Y和Z：

    1. 当X已经锁定mutex，则Y在等待，且Z尝试锁定，会从`S-L`迁移到其自身（路径4）；  
    2. 当X解除锁定、还未唤醒Y时，此时若Z尝试锁定，会从`S--`迁移回`S-L`（路径6）；  
    3. 当X解除锁定、且唤醒Y后（路径9），此时若Z尝试锁定，会从`SW-`迁移到`SWL`（路径11），此时有三条路径：  
        3.1 Y发现mutex已被锁定，而进一步迁移到`S-L`状态（路径18）；  
        3.2 Z迅速解除锁定，状态迁移回`SW-`（路径12），Y尝试锁定，进一步迁移到`S-L`（路径10）；  
        3.3 更多的goroutine尝试锁定，会一直迁移回`SWL`（路径13）。
    4. 当X唤醒Y、且Z尝试锁定成功，则从`-WL`迁移到`SWL`（路径16）。

附录
----

附状态迁移图的[Graphviz](http://www.graphviz.org)源码。

    graph {
        node [shape="circle"];
        edge [dir="forward", len="2.0"];

        rankdir=LR;

        nnn [label="---"];
        nnl [label="--L"];
        swn [label="SW-"];
        snn [label="S--", pos="0,0"];
        snl [label="S-L"];
        nwn [label="-W-"];
        nwl [label="-WL"];
        swl [label="SWL"];

        nnn -- nnl [label="1"];
        nnl -- nnn [label="2"];
        nnl -- snl [label="3"];
        snl -- snl [label="4"];
        snl -- snn [label="5"];
        snn -- snl [label="6"];
        snn -- nwn [label="7"];
        nwn -- nnl [label="8"];
        snn -- swn [label="9"];
        swn -- snl [label="10"];
        swn -- swl [label="11"];
        swl -- swn [label="12"];
        swl -- swl [label="13"];
        nwn -- nwl [label="14"];
        nwl -- nwn [label="15"];
        nwl -- swl [label="16"];
        nwl -- snl [label="17"];
        swl -- snl [label="18"];
    }

