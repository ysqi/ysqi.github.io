
---
date: 2016-12-31T11:33:21+08:00
title: "open-falcon-agent源码学习"
description: ""
disqus_identifier: 1485833601284425406
slug: "open-falcon-agentyuan-ma-xue-xi"
source: "https://segmentfault.com/a/1190000006047609"
tags: 
- go语言 
- golang 
categories:
- 编程语言与开发
---

> 最近学习falcon，看了源码和极客学院的视频解析，画了调用结构、关系，对主要的代码进行了注释

代码地址：[https://github.com/beyondskyw...](https://github.com/beyondskyway/falcon-agent-learn)

标签（空格分隔）： falcon go

------------------------------------------------------------------------

### 监控数据

-   机器性能指标：cpu，mem，网卡，磁盘……

-   业务监控

-   开源软件状态：Nginx，Redis，MySQL

-   snmp采集网络设备指标

### 设计原理

-   自发现采集值

-   不同类型数据采集分不同goroutine

-   进程和端口通过用户配置进行监控

### 配置文件

-   hostname和ip默认留空，agent自动探测

-   hbs和transfer都是配置其rpc地址

-   collector网卡采集前缀

-   ignore为true时取消上报

### 组织结构

-   cron：间隔执行的代码，即定时任务

-   funcs：信息采集

-   g:全局数据结构

-   http：简单的dashboard的server，获取单机监控指标数据

-   plugins：插件处理机制

-   public：静态资源文件

### 心跳机制

-   了解agent、plugin版本信息，方便升级

-   获取监听的进程和端口

-   获取本机执行的插件列表

### 与HBS、Transfer交互

### 调用关系

### 代码解读

-   main入口

<!-- -->

    go cron.InitDataHistory()
    // 上报本机状态
    cron.ReportAgentStatus()
    // 同步插件
    cron.SyncMinePlugins()
    // 同步监控端口、路径、进程和URL
    cron.SyncBuiltinMetrics()
    // 后门调试agent,允许执行shell指令的ip列表
    cron.SyncTrustableIps()
    // 开始数据次采集
    cron.Collect()
    // 启动dashboard server
    go http.Start()

-   ReportAgentStatus：汇报agent本身状态

<!-- -->

    // 判断hbs配置是否正常，正常则上报agent状态
    if g.Config().Heartbeat.Enabled && g.Config().Heartbeat.Addr != "" {
        // 根据配置的interval间隔上报信息
        go reportAgentStatus(time.Duration(g.Config().Heartbeat.Interval) * time.Second)
    }

    func reportAgentStatus(interval time.Duration) {
        for {
            // 获取hostname, 出错则错误赋值给hostname
            hostname, err := g.Hostname()
            if err != nil {
                hostname = fmt.Sprintf("error:%s", err.Error())
            }
            // 请求发送信息
            req := model.AgentReportRequest{
                Hostname:      hostname,
                IP:            g.IP(),
                AgentVersion:  g.VERSION,
                // 通过shell指令获取plugin版本，能否go实现
                PluginVersion: g.GetCurrPluginVersion(),
            }

            var resp model.SimpleRpcResponse
            // 调用rpc接口
            err = g.HbsClient.Call("Agent.ReportStatus", req, &resp)
            if err != nil || resp.Code != 0 {
                log.Println("call Agent.ReportStatus fail:", err, "Request:", req, "Response:", resp)
            }

            time.Sleep(interval)
        }
    }

-   SyncMinePlugins：同步插件

<!-- -->

    func syncMinePlugins() {
        var (
            timestamp  int64 = -1
            pluginDirs []string
        )

        duration := time.Duration(g.Config().Heartbeat.Interval) * time.Second

        for {
            time.Sleep(duration)

            hostname, err := g.Hostname()
            if err != nil {
                continue
            }

            req := model.AgentHeartbeatRequest{
                Hostname: hostname,
            }

            var resp model.AgentPluginsResponse
            // 调用rpc接口,返回plugin
            err = g.HbsClient.Call("Agent.MinePlugins", req, &resp)
            if err != nil {
                log.Println("ERROR:", err)
                continue
            }
            // 保证时间顺序正确
            if resp.Timestamp <= timestamp {
                continue
            }

            pluginDirs = resp.Plugins
            // 存放时间保证最新
            timestamp = resp.Timestamp

            if g.Config().Debug {
                log.Println(&resp)
            }
            // 无插件则清空plugin
            if len(pluginDirs) == 0 {
                plugins.ClearAllPlugins()
            }

            desiredAll := make(map[string]*plugins.Plugin)
            // 读取所有plugin
            for _, p := range pluginDirs {
                underOneDir := plugins.ListPlugins(strings.Trim(p, "/"))
                for k, v := range underOneDir {
                    desiredAll[k] = v
                }
            }
            // 停止不需要的插件,启动增加的插件
            plugins.DelNoUsePlugins(desiredAll)
            plugins.AddNewPlugins(desiredAll)
        }
    }

-   SyncBuiltinMetrics：同步内置metric,包括端口、目录和进程信息

<!-- -->

    func syncBuiltinMetrics() {
        var timestamp int64 = -1
        var checksum string = "nil"

        duration := time.Duration(g.Config().Heartbeat.Interval) * time.Second

        for {
            time.Sleep(duration)
            // 监控端口、目录大小、进程
            var ports = []int64{}
            var paths = []string{}
            var procs = make(map[string]map[int]string)
            var urls = make(map[string]string)

            hostname, err := g.Hostname()
            if err != nil {
                continue
            }

            req := model.AgentHeartbeatRequest{
                Hostname: hostname,
                Checksum: checksum,
            }

            var resp model.BuiltinMetricResponse
            err = g.HbsClient.Call("Agent.BuiltinMetrics", req, &resp)
            if err != nil {
                log.Println("ERROR:", err)
                continue
            }

            if resp.Timestamp <= timestamp {
                continue
            }

            if resp.Checksum == checksum {
                continue
            }

            timestamp = resp.Timestamp
            checksum = resp.Checksum

            for _, metric := range resp.Metrics {

                if metric.Metric == g.URL_CHECK_HEALTH {
                    arr := strings.Split(metric.Tags, ",")
                    if len(arr) != 2 {
                        continue
                    }
                    url := strings.Split(arr[0], "=")
                    if len(url) != 2 {
                        continue
                    }
                    stime := strings.Split(arr[1], "=")
                    if len(stime) != 2 {
                        continue
                    }
                    if _, err := strconv.ParseInt(stime[1], 10, 64); err == nil {
                        urls[url[1]] = stime[1]
                    } else {
                        log.Println("metric ParseInt timeout failed:", err)
                    }
                }
                // {metric: net.port.listen, tags: port=22}
                if metric.Metric == g.NET_PORT_LISTEN {
                    arr := strings.Split(metric.Tags, "=")
                    if len(arr) != 2 {
                        continue
                    }

                    if port, err := strconv.ParseInt(arr[1], 10, 64); err == nil {
                        ports = append(ports, port)
                    } else {
                        log.Println("metrics ParseInt failed:", err)
                    }

                    continue
                }
                // metric: du.bs tags: path=/home/works/logs
                // du -bs /home/works/logs
                if metric.Metric == g.DU_BS {
                    arr := strings.Split(metric.Tags, "=")
                    if len(arr) != 2 {
                        continue
                    }

                    paths = append(paths, strings.TrimSpace(arr[1]))
                    continue
                }
                //mereic: proc.num tags: name=crond
                //或者metric: proc.num tags: cmdline=cfg.json
                if metric.Metric == g.PROC_NUM {
                    arr := strings.Split(metric.Tags, ",")

                    tmpMap := make(map[int]string)

                    for i := 0; i < len(arr); i++ {
                        if strings.HasPrefix(arr[i], "name=") {
                            tmpMap[1] = strings.TrimSpace(arr[i][5:])
                        } else if strings.HasPrefix(arr[i], "cmdline=") {
                            tmpMap[2] = strings.TrimSpace(arr[i][8:])
                        }
                    }

                    procs[metric.Tags] = tmpMap
                }
            }

            g.SetReportUrls(urls)
            g.SetReportPorts(ports)
            g.SetReportProcs(procs)
            g.SetDuPaths(paths)

        }
    }

-   SyncTrustableIps：同步可信IP列表\
    请求获取远程访问执行shell命令的IP白名单，在通过http/run.go调用shell命令是会判断请求IP是否可信

<!-- -->

    func syncTrustableIps() {
        duration := time.Duration(g.Config().Heartbeat.Interval) * time.Second

        for {
            time.Sleep(duration)

            var ips string
            err := g.HbsClient.Call("Agent.TrustableIps", model.NullRpcRequest{}, &ips)
            if err != nil {
                log.Println("ERROR: call Agent.TrustableIps fail", err)
                continue
            }
            // 设置到本地可信IP列表
            g.SetTrustableIps(ips)
        }
    }

-   FuncsAndInterval：拆分不同的采集函数集，方便通过不同goroutine运行

<!-- -->

    // 间隔internal时间执行fs中的函数
    type FuncsAndInterval struct {
        Fs       []func() []*model.MetricValue
        Interval int
    }

    var Mappers []FuncsAndInterval

    // 根据调用指令类型和是否容易被挂起而分类(通过不同的goroutine去执行,避免相互之间的影响)
    func BuildMappers() {
        interval := g.Config().Transfer.Interval
        Mappers = []FuncsAndInterval{
            FuncsAndInterval{
                Fs: []func() []*model.MetricValue{
                    AgentMetrics,
                    CpuMetrics,
                    NetMetrics,
                    KernelMetrics,
                    LoadAvgMetrics,
                    MemMetrics,
                    DiskIOMetrics,
                    IOStatsMetrics,
                    NetstatMetrics,
                    ProcMetrics,
                    UdpMetrics,
                },
                Interval: interval,
            },
            // 容易出问题
            FuncsAndInterval{
                Fs: []func() []*model.MetricValue{
                    DeviceMetrics,
                },
                Interval: interval,
            },
            // 调用相同指令
            FuncsAndInterval{
                Fs: []func() []*model.MetricValue{
                    PortMetrics,
                    SocketStatSummaryMetrics,
                },
                Interval: interval,
            },
            FuncsAndInterval{
                Fs: []func() []*model.MetricValue{
                    DuMetrics,
                },
                Interval: interval,
            },
            FuncsAndInterval{
                Fs: []func() []*model.MetricValue{
                    UrlMetrics,
                },
                Interval: interval,
            },
        }
    }

-   Colleet：配置信息读取，读取Mapper中的FuncsAndInterval，根据func调用采集函数，采集所有信息（**并非先过滤采集项**），从所有采集到的数据中过滤ignore的项，并上报到transfer。

<!-- -->

    func Collect() {
        // 配置信息判断
        if !g.Config().Transfer.Enabled {
            return
        }

        if len(g.Config().Transfer.Addrs) == 0 {
            return
        }
        // 读取mapper中的FuncsAndInterval集,并通过不同的goroutine运行
        for _, v := range funcs.Mappers {
            go collect(int64(v.Interval), v.Fs)
        }
    }

    // 间隔采集信息
    func collect(sec int64, fns []func() []*model.MetricValue) {
        // 启动断续器,间隔执行
        t := time.NewTicker(time.Second * time.Duration(sec)).C
        for {
            <-t

            hostname, err := g.Hostname()
            if err != nil {
                continue
            }

            mvs := []*model.MetricValue{}
            // 读取忽略metric名单
            ignoreMetrics := g.Config().IgnoreMetrics
            // 从funcs的list中取出每个采集函数
            for _, fn := range fns {
                // 执行采集函数
                items := fn()
                if items == nil {
                    continue
                }

                if len(items) == 0 {
                    continue
                }
                // 读取采集数据,根据忽略的metric忽略部分采集数据
                for _, mv := range items {
                    if b, ok := ignoreMetrics[mv.Metric]; ok && b {
                        continue
                    } else {
                        mvs = append(mvs, mv)
                    }
                }
            }
            // 获取上报时间
            now := time.Now().Unix()
            // 设置上报采集项的间隔、agent主机、上报时间
            for j := 0; j < len(mvs); j++ {
                mvs[j].Step = sec
                mvs[j].Endpoint = hostname
                mvs[j].Timestamp = now
            }
            // 调用transfer发送采集数据
            g.SendToTransfer(mvs)
        }
    }

-   采集信息结构

<!-- -->

    type MetricValue struct {
        Endpoint  string      // 主机名
        Metric    string      // 信息标识cpu.idle、mem.memtotal等
        Value     interface{} // 采集结果
        Step      int64       // 该项上报间隔
        Type      string      // GAUGE或COUNTER
        Tags      string      // 配置报警策略
        Timestamp int64       // 此次上报时间
    }

-   采集信息组成metricValue结构

<!-- -->

    func NewMetricValue(metric string, val interface{}, dataType string, tags ...string) *model.MetricValue {
        mv := model.MetricValue{
            Metric: metric,
            Value:  val,
            Type:   dataType,
        }

        size := len(tags)

        if size > 0 {
            mv.Tags = strings.Join(tags, ",")
        }

        return &mv
    }
    // 原值类型
    func GaugeValue(metric string, val interface{}, tags ...string) *model.MetricValue {
        return NewMetricValue(metric, val, "GAUGE", tags...)
    }

    // 计数器类型
    func CounterValue(metric string, val interface{}, tags ...string) *model.MetricValue {
        return NewMetricValue(metric, val, "COUNTER", tags...)
    }

-   rpc组件

<!-- -->

    // 简单封装rpc.Cilent
    type SingleConnRpcClient struct {
        sync.Mutex
        rpcClient *rpc.Client
        RpcServer string
        Timeout   time.Duration
    }

    // 关闭rpc
    func (this *SingleConnRpcClient) close() {
        if this.rpcClient != nil {
            this.rpcClient.Close()
            this.rpcClient = nil
        }
    }

    // 保证rpc存在,为空则重新创建, 如果server宕机, 死循环????
    func (this *SingleConnRpcClient) insureConn() {
        if this.rpcClient != nil {
            return
        }

        var err error
        var retry int = 1

        for {
            if this.rpcClient != nil {
                return
            }
            // 根据timeout和server地址去连接rpc的server
            this.rpcClient, err = net.JsonRpcClient("tcp", this.RpcServer, this.Timeout)
            if err == nil {
                return
            }

            log.Printf("dial %s fail: %v", this.RpcServer, err)

            if retry > 6 {
                retry = 1
            }

            time.Sleep(time.Duration(math.Pow(2.0, float64(retry))) * time.Second)

            retry++
        }
    }

    // rpc client调用hbs函数
    func (this *SingleConnRpcClient) Call(method string, args interface{}, reply interface{}) error {
        // 加锁保证一个agent只与server有一个连接,保证性能
        this.Lock()
        defer this.Unlock()
        // 保证rpc连接可用
        this.insureConn()

        timeout := time.Duration(50 * time.Second)
        done := make(chan error)

        go func() {
            err := this.rpcClient.Call(method, args, reply)
            done <- err
        }()
        // 超时控制
        select {
        case <-time.After(timeout):
            log.Printf("[WARN] rpc call timeout %v => %v", this.rpcClient, this.RpcServer)
            this.close()
        case err := <-done:
            if err != nil {
                this.close()
                return err
            }
        }
        return nil
    }

-   Transfer部件

<!-- -->

    // 定义transfer的rpcClient对应Map, transferClients读写锁
    var (
        TransferClientsLock *sync.RWMutex                   = new(sync.RWMutex)
        TransferClients     map[string]*SingleConnRpcClient = map[string]*SingleConnRpcClient{}
    )

    // 发送数据到随机的transfer
    func SendMetrics(metrics []*model.MetricValue, resp *model.TransferResponse) {
        rand.Seed(time.Now().UnixNano())
        // 随机transferClient发送数据,直到发送成功
        for _, i := range rand.Perm(len(Config().Transfer.Addrs)) {
            addr := Config().Transfer.Addrs[i]
            if _, ok := TransferClients[addr]; !ok {
                initTransferClient(addr)
            }
            if updateMetrics(addr, metrics, resp) {
                break
            }
        }
    }

    // 初始化addr对应的transferClient
    func initTransferClient(addr string) {
        TransferClientsLock.Lock()
        defer TransferClientsLock.Unlock()
        TransferClients[addr] = &SingleConnRpcClient{
            RpcServer: addr,
            Timeout:   time.Duration(Config().Transfer.Timeout) * time.Millisecond,
        }
    }

    // 调用rpc接口发送metric
    func updateMetrics(addr string, metrics []*model.MetricValue, resp *model.TransferResponse) bool {
        TransferClientsLock.RLock()
        defer TransferClientsLock.RUnlock()
        err := TransferClients[addr].Call("Transfer.Update", metrics, resp)
        if err != nil {
            log.Println("call Transfer.Update fail", addr, err)
            return false
        }
        return true
    }

-   采集插件同步

<!-- -->

    // 插件信息: 路径、修改时间、运行周期(来自plugin插件)
    type Plugin struct {
        FilePath string
        MTime    int64
        Cycle    int
    }

    // 插件map和调度器map
    var (
        Plugins              = make(map[string]*Plugin)
        PluginsWithScheduler = make(map[string]*PluginScheduler)
    )

    // 删除不需要的plugin
    func DelNoUsePlugins(newPlugins map[string]*Plugin) {
        for currKey, currPlugin := range Plugins {
            newPlugin, ok := newPlugins[currKey]
            if !ok || currPlugin.MTime != newPlugin.MTime {
                deletePlugin(currKey)
            }
        }
    }

    // 添加同步时增加的plugin
    func AddNewPlugins(newPlugins map[string]*Plugin) {
        for fpath, newPlugin := range newPlugins {
            // 去除重复插件
            if _, ok := Plugins[fpath]; ok && newPlugin.MTime == Plugins[fpath].MTime {
                continue
            }
            // 为新添加的插件新建调度器
            Plugins[fpath] = newPlugin
            sch := NewPluginScheduler(newPlugin)
            PluginsWithScheduler[fpath] = sch
            // 启动plugin调度
            sch.Schedule()
        }
    }

    func ClearAllPlugins() {
        for k := range Plugins {
            deletePlugin(k)
        }
    }

    func deletePlugin(key string) {
        v, ok := PluginsWithScheduler[key]
        if ok {
            // 暂停调度plugin
            v.Stop()
            delete(PluginsWithScheduler, key)
        }
        delete(Plugins, key)
    }

-   插件调度策略

<!-- -->

    // 持续间隔执行plugin
    type PluginScheduler struct {
        Ticker *time.Ticker
        Plugin *Plugin
        Quit   chan struct{}
    }

    // 根据plugin创建新的schedule
    func NewPluginScheduler(p *Plugin) *PluginScheduler {
        scheduler := PluginScheduler{Plugin: p}
        scheduler.Ticker = time.NewTicker(time.Duration(p.Cycle) * time.Second)
        scheduler.Quit = make(chan struct{})
        return &scheduler
    }

    // plugin调度,间隔执行PluginRun,除非收到quit消息
    func (this *PluginScheduler) Schedule() {
        go func() {
            for {
                select {
                case <-this.Ticker.C:
                    PluginRun(this.Plugin)
                case <-this.Quit:
                    this.Ticker.Stop()
                    return
                }
            }
        }()
    }

    // 停止plugin调度
    func (this *PluginScheduler) Stop() {
        close(this.Quit)
    }

    // 执行插件,读取插件运行返回数据并上报transfer
    func PluginRun(plugin *Plugin) {

        timeout := plugin.Cycle*1000 - 500
        fpath := filepath.Join(g.Config().Plugin.Dir, plugin.FilePath)

        if !file.IsExist(fpath) {
            log.Println("no such plugin:", fpath)
            return
        }

        debug := g.Config().Debug
        if debug {
            log.Println(fpath, "running...")
        }

        cmd := exec.Command(fpath)
        var stdout bytes.Buffer
        cmd.Stdout = &stdout
        var stderr bytes.Buffer
        cmd.Stderr = &stderr
        cmd.Start()

        err, isTimeout := sys.CmdRunWithTimeout(cmd, time.Duration(timeout)*time.Millisecond)

        errStr := stderr.String()
        if errStr != "" {
            logFile := filepath.Join(g.Config().Plugin.LogDir, plugin.FilePath+".stderr.log")
            if _, err = file.WriteString(logFile, errStr); err != nil {
                log.Printf("[ERROR] write log to %s fail, error: %s\n", logFile, err)
            }
        }

        if isTimeout {
            // has be killed
            if err == nil && debug {
                log.Println("[INFO] timeout and kill process", fpath, "successfully")
            }

            if err != nil {
                log.Println("[ERROR] kill process", fpath, "occur error:", err)
            }

            return
        }

        if err != nil {
            log.Println("[ERROR] exec plugin", fpath, "fail. error:", err)
            return
        }

        // exec successfully
        data := stdout.Bytes()
        if len(data) == 0 {
            if debug {
                log.Println("[DEBUG] stdout of", fpath, "is blank")
            }
            return
        }

        var metrics []*model.MetricValue
        err = json.Unmarshal(data, &metrics)
        if err != nil {
            log.Printf("[ERROR] json.Unmarshal stdout of %s fail. error:%s stdout: \n%s\n", fpath, err, stdout.String())
            return
        }

        g.SendToTransfer(metrics)
    }

