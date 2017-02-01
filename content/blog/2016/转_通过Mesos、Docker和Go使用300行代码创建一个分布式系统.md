
---
date: 2016-12-31T11:34:23+08:00
title: "通过Mesos、Docker和Go使用300行代码创建一个分布式系统"
description: ""
disqus_identifier: 1485833663542309991
slug: "tong-guo--Mesos、Docker-he--Goshi-yong--300-hang-dai-ma-chuang-jian-yi-ge-fen-bu-shi-ji-tong"
source: "https://segmentfault.com/a/1190000003048009"
tags: 
- golang 
- mesos 
- docker 
topics:
- 编程语言与开发
---

**【摘要】虽然 Docker 和 Mesos 已成为不折不扣的 Buzzwords
，但是对于大部分人来说它们仍然是陌生的，下面我们就一起领略 Mesos
、Docker 和 Go 配合带来的强大破坏力，如何通过 300
行代码打造一个比特币开采系统。**

时下，对于大部分 IT 玩家来说，
[Docker](http://news.oneapm.com/tag/docker/) 和 Mesos
都是熟悉和陌生的：熟悉在于这两个词无疑已成为大家讨论的焦点，而陌生在于这两个技术并未在生产环境得到广泛使用，因此很多人仍然不知道它们究竟有什么优势，或者能干什么。近日，
John Walter 在 Dzone 上撰文 [Creating a Distributed System in 300 Lines
With Mesos, Docker, and
Go](https://dzone.com/articles/creating-a-distributed-system-in-300-lines-with-me)，讲述了
Mesos、Docker 和 Go 配合带来的强大破坏力，本文由
[OneAPM](http://www.oneapm.com/index.html?utm_source=Common&utm_medium=Articles&utm_campaign=TechnicalArticles&from=matefiseco)
工程师编译整理。

诚然，构建一个分布式系统是很困难的，它需要可扩展性、容错性、高可用性、一致性、可伸缩以及高效。为了达到这些目的，分布式系统需要很多复杂的组件以一种复杂的方式协同工作。例如，Apache
Hadoop 在大型集群上并行处理 TB
级别的数据集时，需要依赖有着高容错的文件系统（ HDFS ）来达到高吞吐量。

在之前，每一个新的分布式系统，例如 Hadoop 和 Cassandra
，都需要构建自己的底层架构，包括消息处理、存储、网络、容错性和可伸缩性。庆幸的是，像
Apache Mesos
这样的系统，通过给分布式系统的关键构建模块提供类似操作系统的管理服务，简化了构建和管理分布式系统的任务。Mesos
抽离了 CPU
、存储和其它计算资源，因此开发者开发分布式应用程序时能够将整个数据中心集群当做一台巨型机对待。

构建在 Mesos 上的应用程序被称为框架，它们能解决很多问题： Apache
Spark，一种流行的集群式数据分析工具；Chronos ，一个类似 cron
的具有容错性的分布式 scheduler ，这是两个构建在 Mesos
上的框架的例子。构建框架可以使用多种语言，包括
C++，Go，Python，Java，Haskell 和 Scala。

在分布式系统用例上，比特币开采就是一个很好的例子。比特币将为生成
acceptable hash
的挑战转为验证一块事务的可靠性。可能需要几十年，单台笔记本电脑挖一块可能需要花费超过
150
年。结果是，有许多的“采矿池”允许采矿者将他们的计算资源联合起来以加快挖矿速度。Mesosphere
的一个实习生， Derek
，写了一个比特币开采框架（<https://github.com/derekchiang/Mesos-Bitcoin-Miner>），利用集群资源的优势来做同样的事情。在接下来的内容中，会以他的代码为例。

1 个 Mesos 框架有 1 个 scheduler 和 1 个 executor 组成。scheduler 和
Mesos master 通信并决定运行什么任务，而 executor 运行在 slaves
上面，执行实际任务。大多数的框架实现了自己的 scheduler，并使用 1 个由
Mesos 提供的标准 executors 。当然，框架也可以自己定制 executor
。在这个例子中即会编写定制的 scheduler，并使用标准命令执行器（ executor
）运行包含我们比特币服务的 Docker 镜像。

对这里的 scheduler 来说，需要运行的有两种任务—— one miner server task
and multiple miner worker tasks。 server
会和一个比特币采矿池通信，并给每个 worker 分配 blocks 。Worker
会努力工作，即开采比特币。

任务实际上被封装在 executor 框架中，因此任务运行意味着告诉 Mesos master
在其中一个 slave 上面启动一个 executor
。由于这里使用的是标准命令执行器（executor），因此可以指定任务是二进制可执行文件、bash
脚本或者其他命令。由于 Mesos 支持 Docker，因此在本例中将使用可执行的
Docker 镜像。Docker
是这样一种技术，它允许你将应用程序和它运行时需要的依赖一起打包。

为了在 Mesos 中使用 Docker 镜像，这里需要在 Docker registry
中注册它们的名称：

    const (
        MinerServerDockerImage = "derekchiang/p2pool"
        MinerDaemonDockerImage = "derekchiang/cpuminer"
    )

然后定义一个常量，指定每个任务所需资源：

    const (
        MemPerDaemonTask = 128  // mining shouldn't be    memory-intensive
        MemPerServerTask = 256
        CPUPerServerTask = 1    // a miner server does not use much     CPU
    )

现在定义一个真正的 scheduler ，对其跟踪，并确保其正确运行需要的状态：

    type MinerScheduler struct { 
        // bitcoind RPC credentials
        bitcoindAddr string
        rpcUser      string
        rpcPass      string
        // mutable state
        minerServerRunning  bool
        minerServerHostname string 
        minerServerPort     int    // the port that miner daemons 
                                   // connect to
        // unique task ids
        tasksLaunched        int
        currentDaemonTaskIDs []*mesos.TaskID
    }

这个 scheduler 必须实现下面的接口：

    type Scheduler interface {
        Registered(SchedulerDriver, *mesos.FrameworkID,     *mesos.MasterInfo)
        Reregistered(SchedulerDriver, *mesos.MasterInfo)
        Disconnected(SchedulerDriver)
        ResourceOffers(SchedulerDriver, []*mesos.Offer)
        OfferRescinded(SchedulerDriver, *mesos.OfferID)
        StatusUpdate(SchedulerDriver, *mesos.TaskStatus)
        FrameworkMessage(SchedulerDriver, *mesos.ExecutorID, 
                         *mesos.SlaveID, string)
        SlaveLost(SchedulerDriver, *mesos.SlaveID)
        ExecutorLost(SchedulerDriver, *mesos.ExecutorID,   *mesos.SlaveID, 
                     int)
        Error(SchedulerDriver, string)
    }

现在一起看一个回调函数：

    func (s *MinerScheduler) Registered(_ sched.SchedulerDriver, 
          frameworkId *mesos.FrameworkID, masterInfo *mesos.MasterInfo) {
        log.Infoln("Framework registered with Master ", masterInfo)
    }
    func (s *MinerScheduler) Reregistered(_ sched.SchedulerDriver, 
          masterInfo *mesos.MasterInfo) {
        log.Infoln("Framework Re-Registered with Master ",  masterInfo)
    }
    func (s *MinerScheduler) Disconnected(sched.SchedulerDriver) {
        log.Infoln("Framework disconnected with Master")
    }

**Registered** 在 scheduler 成功向 Mesos master 注册之后被调用。

**Reregistered** 在 scheduler 与 Mesos master
断开连接并且再次注册时被调用，例如，在 master 重启的时候。

**Disconnected** 在 scheduler 与 Mesos master 断开连接时被调用。这个在
master 挂了的时候会发生。

目前为止，这里仅仅在回调函数中打印了日志信息，因为对于一个像这样的简单框架，大多数回调函数可以空在那里。然而，下一个回调函数就是每一个框架的核心，必须要认真的编写。

**ResourceOffers** 在 scheduler 从 master 那里得到一个 offer
的时候被调用。每一个 offer
包含一个集群上可以给框架使用的资源列表。资源通常包括 CPU
、内存、端口和磁盘。一个框架可以使用它提供的一些资源、所有资源或者一点资源都不给用。

针对每一个 offer
，现在期望聚集所有的提供的资源并决定是否需要发布一个新的 server
任务或者一个新的 worker 任务。这里可以向每个 offer
发送尽可能多的任务以测试最大容量，但是由于开采比特币是依赖 CPU
的，所以这里每个 offer 运行一个开采者任务并使用所有可用的 CPU 资源。

    for i, offer := range offers {
        // … Gather resource being offered and do setup
        if !s.minerServerRunning && mems >= MemPerServerTask &&
                cpus >= CPUPerServerTask && ports >= 2 {
            // … Launch a server task since no server is running and     we 
            // have resources to launch it.
        } else if s.minerServerRunning && mems >= MemPerDaemonTask {
            // … Launch a miner since a server is running and we have     mem 
            // to launch one.
        }
    }

针对每个任务都需要创建一个对应的 TaskInfo message
，它包含了运行这个任务需要的信息。

    s.tasksLaunched++
    taskID = &mesos.TaskID {
        Value: proto.String("miner-server-" + 
                            strconv.Itoa(s.tasksLaunched)),
    }

Task IDs 由框架决定，并且每个框架必须是唯一的。

    containerType := mesos.ContainerInfo_DOCKER
    task = &mesos.TaskInfo {
        Name: proto.String("task-" + taskID.GetValue()),
        TaskId: taskID,
        SlaveId: offer.SlaveId,
        Container: &mesos.ContainerInfo {
            Type: &containerType,
            Docker: &mesos.ContainerInfo_DockerInfo {
                Image: proto.String(MinerServerDockerImage),
            },
        },
        Command: &mesos.CommandInfo {
            Shell: proto.Bool(false),
            Arguments: []string {
                // these arguments will be passed to run_p2pool.py
                "--bitcoind-address", s.bitcoindAddr,
                "--p2pool-port", strconv.Itoa(int(p2poolPort)),
                "-w", strconv.Itoa(int(workerPort)),
                s.rpcUser, s.rpcPass,
            },
        },
        Resources: []*mesos.Resource {
            util.NewScalarResource("cpus", CPUPerServerTask),
            util.NewScalarResource("mem", MemPerServerTask),
        },
    }

TaskInfo message 指定了一些关于任务的重要元数据信息，它允许 Mesos
节点运行 Docker 容器，特别会指定 name、task ID、container information
以及一些需要给容器传递的参数。这里也会指定任务需要的资源。

现在 TaskInfo 已经被构建好，因此任务可以这样运行：

    driver.LaunchTasks([]*mesos.OfferID{offer.Id}, tasks,     &mesos.Filters{RefuseSeconds: proto.Float64(1)})

在框架中，需要处理的最后一件事情是当开采者 server
关闭时会发生什么。这里可以利用 StatusUpdate 函数来处理。

在一个任务的生命周期中，针对不同的阶段有不同类型的状态更新。对这个框架来说，想要确保的是如果开采者
server 由于某种原因失败，系统会 Kill 所有开采者 worker
以避免浪费资源。这里是相关的代码：

    if strings.Contains(status.GetTaskId().GetValue(), "server") &&
        (status.GetState() == mesos.TaskState_TASK_LOST ||
            status.GetState() == mesos.TaskState_TASK_KILLED ||
            status.GetState() == mesos.TaskState_TASK_FINISHED ||
            status.GetState() == mesos.TaskState_TASK_ERROR ||
            status.GetState() == mesos.TaskState_TASK_FAILED) {
        s.minerServerRunning = false
        // kill all tasks
        for _, taskID := range s.currentDaemonTaskIDs {
            _, err := driver.KillTask(taskID)
            if err != nil {
                log.Errorf("Failed to kill task %s", taskID)
            }
        }
        s.currentDaemonTaskIDs = make([]*mesos.TaskID, 0)
    }

万事大吉！通过努力，这里在 Apache Mesos
上建立一个正常工作的分布式比特币开采框架，它只用了大约 300 行 GO
代码。这证明了使用 Mesos 框架的 API 编写分布式系统是多么快速和简单。

原文链接：[Creating a Distributed System in 300 Lines With Mesos,
Docker, and
Go](https://dzone.com/articles/creating-a-distributed-system-in-300-lines-with-me)

**本文由[OneAPM](http://www.oneapm.com/?hmsr=media&hmmd=&hmpl=&hmkw=&hmci=)工程师编译
，想阅读更多技术文章，请访问[OneAPM官方技术博客](http://code.oneapm.com/?hmsr=media&hmmd=&hmpl=&hmkw=&hmci=)。**

