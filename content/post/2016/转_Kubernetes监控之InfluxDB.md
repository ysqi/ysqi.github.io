
---
date: 2016-12-31T11:32:45+08:00
title: "Kubernetes监控之InfluxDB"
description: ""
disqus_identifier: 1485833565144852694
slug: "Kubernetesjian-kong-zhi-InfluxDB"
source: "https://segmentfault.com/a/1190000007702597"
tags: 
- 监控 
- 数据库 
- influxdb 
- kubernetes 
- golang 
categories:
- 编程语言与开发
---

什么是InfluxDB？
----------------

### InfluxDB介绍

InfluxDB是一款用Go语言编写的开源分布式时序、事件和指标数据库，无需外部依赖。\
该数据库现在主要用于存储涉及大量的时间戳数据，如DevOps监控数据，APP
metrics, loT传感器数据和实时分析数据。\
InfluxDB特征：

-   无结构(无模式)：可以是任意数量的列

-   可以设置metric的保存时间

-   支持与时间有关的相关函数(如min、max、sum、count、mean、median等)，方便统计

-   支持存储策略:可以用于数据的删改。(influxDB没有提供数据的删除与修改方法)

-   支持连续查询:是数据库中自动定时启动的一组语句，和存储策略搭配可以降低InfluxDB的系统占用量。

-   原生的HTTP支持，内置HTTP API

-   支持类似sql语法

-   支持设置数据在集群中的副本数

-   支持定期采样数据，写入另外的measurement，方便分粒度存储数据。

-   自带web管理界面，方便使用(登入方式：<http://%3C>
    InfluxDB-IP &gt;:8083)

### 关键概念

InfluxDB关键概念列表：

database

field key

field set

field value

measurement

point

retention policy

series

tag key

tag set

tag value

timestamp

下面举个例子进行概念介绍：\
我们虚拟一组数据，其中有一张数据表(**measurement**)为census，该表记录了由两个科学家(langstroth和perpetua)在两个不同的位置(1和2)，统计了butterflies和honeybees的数据，时间段是2015-08-18
00: 00:00 -- 2015-08-18 06: 12:00.
我们假设这些数据属于叫my\_database的数据库(**database**)，且该数据存储在autogen的存储策略(**retention
policy**)中。\
数据展示如下：

    name: census
    ---------------------
    time                    butterflies     honeybees     location     scientist
    2015-08-18T00:00:00Z      12             23              1         langstroth
    2015-08-18T00:00:00Z      1              30              1         perpetua
    2015-08-18T00:06:00Z      11             28              1         langstroth
    2015-08-18T00:06:00Z      3              28              1         perpetua
    2015-08-18T05:54:00Z      2              11              2         langstroth
    2015-08-18T06:00:00Z      1              10              2         langstroth
    2015-08-18T06:06:00Z      8              23              2         perpetua
    2015-08-18T06:12:00Z      7              22              2         perpetua

我们针对数据来进行概念分析：\
InfluxDB是时序数据库，所以怎么都绕不开时间,第一纵列time存储着时间戳，而时间戳是与数据进行关联，这样才能将时间和数据进行展示。\
接下去两纵列(butterflies和honeybees)，称为**Fields**。**Fields由field
keys和field values组成**。butterflies和honeybees两个字符串就是field
keys；而butterflies这个field key对应的field values就是12 -- 7,
honeybees这个field key对应的field values就是23 -- 22。\
Field
values就是你的数据，它们可以是string、float、int或者bool等。因为influxdb是时序数据库，所以field
values总是要和timestamp关联。

**field set**是在数据层之上应用概念，由field key和field value组成了field
set，如这里有8组field set数据：

-   butterflies = 12 honeybees = 23

-   butterflies = 1 honeybees = 30

-   butterflies = 11 honeybees = 28

-   butterflies = 3 honeybees = 28

-   butterflies = 2 honeybees = 11

-   butterflies = 1 honeybees = 10

-   butterflies = 8 honeybees = 23

-   butterflies = 7 honeybees = 22

field是InfluxDB的必要结构，但也需要注意field是没有索引的。

剩下的两个纵列是location和scientist，它们是**tags**。**Tags也是由键值对(tag
keys和tag values)组成。**这里的tag
keys是字符串location和scientist；location 这个tag key有两个tag values:
1和2；scientist这个tag key也有两个tag values：perpetua和langstroth。\
**tag set**也是数据之上的概念，是不同的tag key-value组合，这里有4组tag
sets数据：

-   location = 1, scientist = langstroth

-   location = 2, scientist = langstroth

-   location = 1, scientist = perpetua

-   location = 2, scientist = perpetua

Tags是可选的参数，也就是说你存储的数据结构中不一定非要带tags，但是它非常好用，因为可以索引。一般都会通过tags来查询数据会快很多。

**measurement**包含了tags、fields和time，就类似于传统数据库的表。一个measurement可以属于不同的retention
policy(存储策略)，存储策略描述了InfluxDB怎么去保持数据(DURATION)，需要在集群中存储多少份数据副本(REPLICATION)。\
示例中的数据都属于census这个measurement，而该measurement又属于autogen这个存储策略。InfluxDB一般都会创建一个default存储策略，它有无限长的持续时间和等于1的副本数。

我们了解过了measurements、tag sets和retention
policies的概念后，是时候该知道**series**了。\
在同一个database中，series由retention policy、measurement、tag
sets三部分组成，在我们上面的数据中有如下4个series：

  Arbitrary series number   Retention policy   Measurement   Tag set
  ------------------------- ------------------ ------------- -------------------------------------
  series 1                  autogen            census        location = 1,scientist = langstroth
  series 2                  autogen            census        location = 2,scientist = langstroth
  series 3                  autogen            census        location = 1,scientist = perpetua
  series 4                  autogen            census        location = 2,scientist = perpetua

同一个Series的数据在物理上会按照时间顺序排列存储在一起。\
Series的key为measurement + 所有tags的序列化字符串。\
代码结构如下：

    tyep Series struct {
        mu           sync.RWMutex
        Key          string
        Tags         map[string]string  
        id           uint64
        measurement  *Measurement
    }

介绍完Series后，就可以解释point了。point是在一个series中有相同时间戳的field
set，也可以理解如表里的一行数据。示例中一个Point：

    name: census
    -----------------
    time                   butterflies     honeybees     location     scientist
    2015-08-18T00:00:00Z        1              30           1         perpetua

上例中，series由retention policy(autogen), measurement(census)和tag
set(location=1,scientist=perpetua)进行定义。而这个point的时间戳则是2015-08-18T
00: 00: 00Z。

InfluxDB Database可以有多个users、continuous queries、retention
policy、measurement。因为InfluxDB是一个结构化的数据库，我们可以轻松的去新增measurements、tags、fields。

### 高级概念

#### [**Retention Policy**](https://docs.influxdata.com/influxdb/v1.1/query_language/database_management/#retention-policy-management)

之前讲关键性概念时有简单介绍了RP，这里会进行较详细的介绍。\
InfluxDB的数据保留策略(RP)是用来定义数据在数据库中存放的时间，或者定义保存某个期间的数据。\
RP在InfluxDB中是比较重要的概念，因为InfluxDB本身是没有提供数据的删除操作，所以需要通过定义RP来控制数据量的问题。\
(一个数据库可以有多个RP，但是每个RP必须是独一无二的。)

在具体介绍RP之前，先介绍下另外一个跟RP相关的基础概念(**shard**)。\
**shard:**\
每个RP下面会存在很多shard，每个shard都存储了实际编码和压缩数据，并且不重复。例如你在创建RP时指定了shard
duration为1h，那么7--8点存入shard\_group0,8--9点就会存入shard\_group1中。所以shard才是真实存储InfluxDB数据的地方。\
每个shard都属于唯一一个shard
group，一个group中会有多个shard；而每个shard包含一组特定的series；所有的points都落在给定的series中，而series是都落在给定的shard
group中；

> 问题1：每个shard
> group指定了一段时间区域，而且其中有多个shard；每个shard包含一组特定的series。那么shard中存的数据是怎么区分的？series是由RP、meansurement、tags组成，那么shard的区分是根据tags？？

**shard duration:**\
shard duration决定了每个shard
group存放数据的时间区域。这段时间是在定义RP时由"SHARD
DURATION"字段决定。\
例如你创建RP时指定了SHARD DURATION为1w,那么每个shard
group的时间跨度就为1w，它将包含所有在这一周时间戳内的points。

OK，大概了解了shard之后，继续回到Retention Policy。

当你创建一个数据库时，InfluxDB会自动给你创建一个叫"autogen"的retention
Policy，这个RP的数据保留时间是无限。\
1.创建RP语法：

    CREATE RETETION POLICY {rp_name} ON {database_name} DURATION {duration} REPLICATION {n} [SHARD DURATION {duration}] [DEFAULT]

注：\
DURATION: 用于描述数据保留时间。可设置的时间区间是1h -- INF(无穷大)。\
REPLICATION: 用于指定数据的备份数量，n是表示数据节点的数量。\
SHARD DURATION: 用于指定shard
group的时间区域，这个字段的duration是不支持INF的。默认情况下，shard
group的duration由RP的duration决定。

  Retention Policy's DURATION       Shard Group Duration
  --------------------------------- ----------------------
  &lt; 2 days                       1h
  &gt;= 2 days and &lt;= 6 mouths   1day
  &gt; 6 mouths                     7days

DEFAULT:
可选参数，用于指定使用新的RP来作为数据库的默认RP。(具体新在哪？需要进一步查看)

2.修改RP语法：

    ALTER RETENTION POLICY {rp_name} ON {database_name} DURATION {duration} REPLICATION {n} SHARD DURATION {duration} DEFAULT

注：\
后面的参数字段都一样，主要差别就在于关键字段：ALTER RETENTION POLICY

3.删除RP语法：

    DROP RETENTION POLICY {rp_name} ON {database_name}

注：\
即使你企图去删除一个不存在的rp，命令返回值也是空，不会返回一个错误码。

#### [**Continuous Queries**](https://docs.influxdata.com/influxdb/v1.1/query_language/continuous_queries/)

之前我们介绍了数据保存策略，数据超过保存策略里指定的时间之后，就会被删除。但我们不想完全删除这些数据，比如我们想把每秒的监控数据至少保留成每小时，就需要用到连续查询(Continuous
Queries)功能。\
连续查询主要用在将数据归档，以降低系统空间的占用率，但这主要是以降低数据精度为代价。\
**基本语法：**

    CREATE CONTINUOUS QUERY {cq_name} ON {database_name}
    BEGIN
        {cq_query}
    END
    注：cq_name表示创建的Continuous query的名字；database_name表示要操作的数据库。

    cq_query是操作函数，如下：
    SELECT {function[s]} INTO {destnation_measurement} FROM {measurement} [WHERE {stuff}] GROUP BY time({interval})[,{tag_key[s]}]
    注：destnation_measurement表示新生成的数据存放的表；measurement表示数据查询的表；
    GROUP BY time表示采样分析的数据时间，比如设置1h，如果当前是17:00,那么需要计算的数据时间就是16:00 -- 16：59。

**例子1： 自动降低精度来采样数据**

    CREATE CONTINUOUS QUERY "cq_basic" ON "transportation"
    BEGIN
        SELECT mean("passengers") INTO "average_passengers" FROM "bus_data" GROUP BY time(1h)
    END

    查看结果：
    > SELECT * FROM "average_passengers"
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T07:00:00Z   7
    2016-08-28T08:00:00Z   13.75

连续查询（cq\_basic）通过在数据库"transportation"中的"bus\_data"表，计算每小时平均的旅客数，然后在该数据库中新建"average\_passengers"表，并将数据存入该表中。该cq\_basic每小时执行一遍，然后将每个小时的point写入表中。

**例子2：自动降低精度来采样数据，并将数据存入另外一个Retention
Policy(RP)**

    CREATE CONTINUOUS QUERY "cq_basic_rp" ON "transportation"
    BEGIN
        SELECT mean("passengers") INTO "transportation"."three_weeks"."average_passengers" FROM "bus_data" GROUP BY time(1h)
    END

    查看结果：
    > SELECT * FROM "transportation"."three_weeks"."average_passengers"
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T07:00:00Z   7
    2016-08-28T08:00:00Z   13.75

连续查询（cq\_basic\_rp）通过在数据库"transportation"中的"bus\_data"表，计算每小时平均的旅客数，然后将数据存入transportation数据库中的three\_weeks(RP)的average\_passengers表中。该cq\_basic\_rp每小时执行一遍，然后将每个小时的point写入表中。

**例子3：采用通配符，自动降低精度采样数据**

    CREATE CONTINUOUS QUERY "cq_basic_br" ON "transportation"
    BEGIN
        SELECT mean(*) INTO "dowmsample_transportation"."autogen".:MEASUREMENT FROM /.*/ GROUP BY time(30m),*
    END

    查看结果：
    > SELECT * FROM "downsample_transportation"."autogen"."bus_data"
    name: bus_data
    --------------
    time                   mean_complaints   mean_passengers
    2016-08-28T07:00:00Z   9                 6.5
    2016-08-28T07:30:00Z   9                 7.5
    2016-08-28T08:00:00Z   8                 11.5
    2016-08-28T08:30:00Z   7                 16

连续查询（cq\_basic\_br），计算数据库(transportation)中每张表(这里只有一张表"bus\_data")，每30分钟平均的**旅客数和投诉量**，然后将数据存入downsample\_transportation数据库中的autogen(RP)中。该cq\_basic\_br每30分钟执行一遍，然后将每个小时的point写入表中。

**例子4：配置CQ的时间偏移，来采集数据：**

    CREATE CONTINUOUS QUERY "cq_basic_offset" ON "transportation"
    BEGIN
        SELECT mean("passengers") INTO "average_passengers" FROM "bus_data" GROUP BY time(1h,15m)
    END

    查看结果：
    > SELECT * FROM "average_passengers"
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T07:15:00Z   7.75      //注意时间是从7:15 -- 8:15
    2016-08-28T08:15:00Z   16.75

该CQ(cq\_basic\_offset)，设置了每整点往后偏移15分钟，再进行每小时的平均值计算。比如会将8
: 15--9: 15，来代替8: 00--9: 00。

**高级语法：**

    CREATE CONTINUOUS QUERY {cq_name} ON {database_name}
    RESAMPLE EVERY {val1} FOR {val2}
    BEGIN
        {cq_query}
    END

    注意： cq_name、database_name、cq_query和之前的基本语法都一致。
    EVERY后面带的时间，表示每val1点时间就触发一次数据采样，而数据的区间是和cq_query、FOR有关。在这段时间内每val1点时间再采集一次。比如cq_query设置1h，val1设置为30m,表示在1h内会有两次数据计算。比如8点--9点之间就会有两次数据的计算，第一次计算是8:30触发的，计算的区间是8:00--8:30，第二次计算是9:00触发的，计算的区间是8:00--9:00。在目的数据库中，默认第二次的计算结果会覆盖第一次的计算结果。
    FOR后面带的时间，表示修改了cq_query计算的数据区间，比如cq_query时间设置为30m，val2设置的是1h。那么cq每30m会触发一次数据计算，计算的区间是(now-1h)--now。

示例数据： 给下面的例子使用

    name: bus_data
    --------------
    time                   passengers
    2016-08-28T06:30:00Z   2
    2016-08-28T06:45:00Z   4
    2016-08-28T07:00:00Z   5
    2016-08-28T07:15:00Z   8
    2016-08-28T07:30:00Z   8
    2016-08-28T07:45:00Z   7
    2016-08-28T08:00:00Z   8
    2016-08-28T08:15:00Z   15
    2016-08-28T08:30:00Z   15
    2016-08-28T08:45:00Z   17
    2016-08-28T09:00:00Z   20

**例子1：配置执行间隔**

    CREATE CONTINUOUS QUERY "cq_advanced_every" ON "transportation"
    RESAMPLE EVERY 30m
    BEGIN
      SELECT mean("passengers") INTO "average_passengers" FROM "bus_data" GROUP BY time(1h)
    END

    中间的执行过程：
    At 8:00, cq_advanced_every executes a query with the time range WHERE time >= '7:00' AND time < '8:00'.
    cq_advanced_every writes one point to the average_passengers measurement:
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T07:00:00Z   7

    At 8:30, cq_advanced_every executes a query with the time range WHERE time >= '8:00' AND time < '9:00'.
    cq_advanced_every writes one point to the average_passengers measurement:
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T08:00:00Z   12.6667

    At 9:00, cq_advanced_every executes a query with the time range WHERE time >= '8:00' AND time < '9:00'.
    cq_advanced_every writes one point to the average_passengers measurement:
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T08:00:00Z   13.75

    查看结果：
    > SELECT * FROM "average_passengers"
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T07:00:00Z   7
    2016-08-28T08:00:00Z   13.75

cq\_advanced\_every在8点--9点执行了两次。第一次8:30触发，因为cq\_query设置了1h,所以数据区间是8:
00--9: 00,但因为是在8:30触发的，8: 30--9:
00的数据还没产生呢，所以实际采集的数据区间是在8: 00--8: 30,即数据(8, 15,
15), 计算的平均值为12.6667；第二次9:00触发，计算的区间是8: 00--9:
00，即数据(8, 15, 15, 17)，计算的平均值为13.75.

**例子2：配置重采样的时间区间**

    CREATE CONTINUOUS QUERY "cq_advanced_for" ON "transportation"
    RESAMPLE FOR 1h
    BEGIN
      SELECT mean("passengers") INTO "average_passengers" FROM "bus_data" GROUP BY time(30m)
    END

    采样过程：
    At 8:00 cq_advanced_for executes a query with the time range WHERE time >= '7:00' AND time < '8:00'.
    cq_advanced_for writes two points to the average_passengers measurement:
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T07:00:00Z   6.5
    2016-08-28T07:30:00Z   7.5

    At 8:30 cq_advanced_for executes a query with the time range WHERE time >= '7:30' AND time < '8:30'.
    cq_advanced_for writes two points to the average_passengers measurement:
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T07:30:00Z   7.5
    2016-08-28T08:00:00Z   11.5

    At 9:00 cq_advanced_for executes a query with the time range WHERE time >= '8:00' AND time < '9:00'.
    cq_advanced_for writes two points to the average_passengers measurement:
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T08:00:00Z   11.5
    2016-08-28T08:30:00Z   16

    结果查询：
    > SELECT * FROM "average_passengers"
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T07:00:00Z   6.5
    2016-08-28T07:30:00Z   7.5
    2016-08-28T08:00:00Z   11.5
    2016-08-28T08:30:00Z   16

该cq\_advanced\_for，每30m重采样一次，采样的区间是(now-1h -- now),
也就是每触发一次执行，就会进行两次计算。因为采样的区间是1h，而需要计算的是每30m的平均值。

**例子3：配置cq的执行区间和时间范围**

    CREATE CONTINUOUS QUERY "cq_advanced_every_for" ON "transportation"
    RESAMPLE EVERY 1h FOR 90m
    BEGIN
      SELECT mean("passengers") INTO "average_passengers" FROM "bus_data" GROUP BY time(30m)
    END

    采样过程：
    At 8:00 cq_advanced_every_for executes a query with the time range WHERE time >= '6:30' AND time < '8:00'.
    cq_advanced_every_for writes three points to the average_passengers measurement:
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T06:30:00Z   3
    2016-08-28T07:00:00Z   6.5
    2016-08-28T07:30:00Z   7.5

    At 9:00 cq_advanced_every_for executes a query with the time range WHERE time >= '7:30' AND time < '9:00'.
    cq_advanced_every_for writes three points to the average_passengers measurement:
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T07:30:00Z   7.5
    2016-08-28T08:00:00Z   11.5
    2016-08-28T08:30:00Z   16

    结果查询：
    > SELECT * FROM "average_passengers"
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T06:30:00Z   3
    2016-08-28T07:00:00Z   6.5
    2016-08-28T07:30:00Z   7.5
    2016-08-28T08:00:00Z   11.5
    2016-08-28T08:30:00Z   16

该cq\_advanced\_every\_for，需要计算30m的平均值，每1小时触发一次cq执行,采样的数据区间是90m，所以每触发一次就会计算3次平均值。

**例子4：配置CQ的采样时间区间，并且填充空结果**

    CREATE CONTINUOUS QUERY "cq_advanced_for_fill" ON "transportation"
    RESAMPLE FOR 2h
    BEGIN
      SELECT mean("passengers") INTO "average_passengers" FROM "bus_data" GROUP BY time(1h) fill(1000)
    END

    采样过程：
    At 6:00, cq_advanced_for_fill executes a query with the time range WHERE time >= '4:00' AND time < '6:00'.
    cq_advanced_for_fill writes nothing to average_passengers; bus_data has no data that fall within that time range. 

    At 7:00, cq_advanced_for_fill executes a query with the time range WHERE time >= '5:00' AND time < '7:00'.
    cq_advanced_for_fill writes two points to average_passengers:
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T05:00:00Z   1000          <------ fill(1000)
    2016-08-28T06:00:00Z   3             <------ average of 2 and 4

    […] 

    At 11:00, cq_advanced_for_fill executes a query with the time range WHERE time >= '9:00' AND time < '11:00'.
    cq_advanced_for_fill writes two points to average_passengers:
    name: average_passengers
    ------------------------
    2016-08-28T09:00:00Z   20            <------ average of 20
    2016-08-28T10:00:00Z   1000          <------ fill(1000)     

    At 12:00, cq_advanced_for_fill executes a query with the time range WHERE time >= '10:00' AND time < '12:00'.
    cq_advanced_for_fill writes nothing to average_passengers; bus_data has no data that fall within that time range.

    结果查询：
    > SELECT * FROM "average_passengers"
    name: average_passengers
    ------------------------
    time                   mean
    2016-08-28T05:00:00Z   1000
    2016-08-28T06:00:00Z   3
    2016-08-28T07:00:00Z   7
    2016-08-28T08:00:00Z   13.75
    2016-08-28T09:00:00Z   20
    2016-08-28T10:00:00Z   1000

该cq\_advcanced\_for\_fill，增加了空数据区的默认值填充，使用fill(value)来实现。

**连续查询使用案例：**\
1.实现重采样和数据保留：\
使用CQ和retention policy配合达到该功能。可以降低数据库存储压力。

2.预先计算来解决费时的查询：\
CQ会自动进行重采样，将高精度的数据转换为低精度的数据。低精度的数据查询会耗费更少的资源和时间。

3.替代HAVING条款：\
InfluxDB不支持HAVING字段，需要使用CQ+别的命令来实现替换。\
例子：

    SELECT mean("bees") FROM "farm" GROUP BY time(30m) HAVING mean("bees") > 20

以上的命令，InfluxDB不支持。其实就是需要实现采集30m的平均值，然后取那些大于20的值。\
InfluxDB的替代方案：

-   先创建CQ：

<!-- -->

    CREATE CONTINUOUS QUERY "bee_cq" ON "mydb" 
    BEGIN
        SELECT mean("bees") AS "mean_bees" INTO "aggregate_bees" FROM "farm" GROUP BY time(30m) 
    END

该创建的CQ，每30m进行bees的平均值计算，并将结果写入aggregate\_bees表中的mean\_bees
field中。

-   查询CQ结果：\
    这一步就是需要运行HAVING mean("bees") &gt;
    20这条命令。InfluxDB命令使用如下：

<!-- -->

    SELECT "mean_bees" FROM "aggregate_bees" WHERE "mean_bees" > 20

4.替代内嵌函数：\
InfluxDB不支持内嵌函数，比如：

    SELECT mean(count("bees")) FROM "farm" GROUP BY time(30m)

替换上述方案：

-   创建CQ:

<!-- -->

    CREATE CONTINUOUS QUERY "bee_cq" ON "mydb" 
    BEGIN
        SELECT count("bees") AS "count_bees" INTO "aggregate_bees" FROM "farm" GROUP BY time(30m) 
    END

-   查询CQ结果：\
    这一步就是需要执行mean(\[...\])这条命令，其实就是计算某段区间的count("bees")平均值,如下：

<!-- -->

    SELECT mean("count_bees") FROM "aggregate_bees" WHERE time >= {start_time} AND time <= {end_time}

Kapacitor是InfluxData的数据处理引擎，它可以达到CQ一样的功能。参考[**HERE**](https://docs.influxdata.com/kapacitor/v1.1/examples/continuous_queries/)

InfluxDB使用
------------

### 数据库配置

参考[**Here**](https://docs.influxdata.com/influxdb/v1.1/administration/config/#meta)

### Database

1.查询：

    SHOW DATABASES 

2.创建：

    CREATE DATABASE {database_name} [WITH [DURATION <duration>] [REPLICATION <n>] [SHARD DURATION <duration>] [NAME <retention-policy-name>]]
    注：WITH带的这段属性，就是Retention Policy的，可以参考它。

3.删除：

    DROP DATABASE {database_name}

### RETENTION POLICY

1.查询：

    SHOW RETETION POLICIES

2.创建：

    CREATE RETENTION POLICY {retention_policy_name} ON {database_name} DURATION {duration} REPLICATION {n} [SHARD DURATION {duration}] [DEFAULT]

3.修改：

    ALTER RETENTION POLICY {rp_name} ON {database_name} DURATION {duration} REPLICATION {n} SHARD DURATION {duration} DEFAULT

4.删除：

    DROP RETENTION POLICY {rp_name} ON {database_name}

### CONTINUOUS QUERY:

1.查询：

    SHOW CONTINUOUS QUERY

2.创建：

    参考之前的例子，介绍了较多的创建方式。

3.删除：

    DROP CONTINUOUS QUERY {cq_name} ON {database_name}

举了部分例子，具体的可以再查看官方资料。

API
---

InfluxDB
API提供了较简单的方式用于数据库交互。该API使用了HTTP的方式，并以JSON格式进行返回。\
下面对API进行介绍：

### 支持的Endpoints

  Endpoint   描述
  ---------- -------------------------------------------------
  /ping      使用/ping用于检查InfluxDB的状态或者版本信息
  /query     使用/query用于查询数据，管理数据库、rp、users等
  /write     使用/write去写数据到数据库中

### /ping

/ping支持GET和HEAD，都可用于获取指定信息。\
定义：

-   GET <http://localhost:8086/ping>

-   HEAD <http://localhost:8086/ping>

示例：\
获取InfluxDB版本信息：

    $ curl -sl -I http://localhost:8086/ping
    HTTP/1.1 204 No Content
    Request-Id: 245a330d-baba-11e6-8098-000000000000
    X-Influxdb-Version: 0.9.4.1
    Date: Mon, 05 Dec 2016 07:12:11 GMT

### /query

/query支持GET和POST的HTTP请求。可用于查询数据和管理数据库、rp、users。\
**定义：**

-   GET <http://localhost:8086/query>

-   POST <http://localhost:8086/query>

**用法说明：**

  ------------------------------
  动作   查询类型
  ------ -----------------------
  GET    用于所有数据的查询：\
         SELECT \*\
         SHOW

  POST   支持的动作如下：\
         ALTER\
         CREATE\
         DELETE\
         DROP\
         GRANT\
         KILL\
         REVOKE
  ------------------------------

> 只有SELECT特殊点，支持INTO字段

**示例：**\
1.使用SELECT查询数据：

    $ curl -G 'http://localhost:8086/query?db=mydb' --data-urlencode 'q=SELECT * FROM "mymeas"'

    {"results":[{"series":[{"name":"mymeas","columns":["time","myfield","mytag1","mytag2"],"values":[["2016-05-20T21:30:00Z",12,"1",null],["2016-05-20T21:30:20Z",11,"2",null],["2016-05-20T21:30:40Z",18,null,"1"],["2016-05-20T21:31:00Z",19,null,"3"]]}]}]}

再使用额外的INTO字段：

    $ curl -XPOST 'http://localhost:8086/query?db=mydb' --data-urlencode 'q=SELECT * INTO "newmeas" FROM "mymeas"'

    {"results":[{"series":[{"name":"result","columns":["time","written"],"values":[["1970-01-01T00:00:00Z",4]]}]}]}

2.创建数据库：

    $ curl -XPOST 'http://localhost:8086/query' --data-urlencode 'q=CREATE DATABASE "mydb"'

    {"results":[{}]}

**Query参数说明：**

  参数                                       是否可选   描述
  ------------------------------------------ ---------- ---------------------------------------------------------------------------------------------------------------------
  chunked=\[true or {number\_of\_points}\]   可选       返回批量的points信息，以代替单个响应。设置成true，InfluxDB返回一批series或者10000个points；或者设置对应的points数量
  db={db\_name}                              必选       设置数据库名
  epoch=\[h,m,s,ms,u,ns\]                    可选       指定时间戳的精度，默认是ns
  p={password}                               可选       如果设置了认证，则需要用户密码
  pretty=true                                可选       优化输出格式，设置之后会议json格式进行输出，利于调试
  rp={rp\_name}                              可选       设置查询的rp。如果没有设置，则查询默认的rp
  u={username}                               可选       如果设置了认证，则需要用户密码

示例1：使用http认证来创建数据库：

    $ curl -XPOST 'http://localhost:8086/query?u=myusername&p=mypassword' --data-urlencode 'q=CREATE DATABASE "mydb"'

    {"results":[{}]}

示例2：使用基础认证来创建数据库：

    $ curl -XPOST -u myusername:mypassword 'http://localhost:8086/query' --data-urlencode 'q=CREATE DATABASE "mydb"'

    {"results":[{}]}

**数据请求体：**

    --data-urlencode 'q=< influxDB query >'

-   可支持多条请求命令： 需要使用分号(;)，来进行命令分隔

-   可支持导入文件的格式进行查询：
    如果文件中使用了多条请求命令，则也需要使用分号(;)进行分隔

        语法：
        curl -F "q=@<path_to_file>" -F "async=true" http://localhost:8086/query

-   以CSV的格式返回请求结果：

        语法：
        curl -H "Accept: application/csv" -G 'http://localhost:8086/query [...]

-   支持绑定参数：\
    该API支持使用WHERE绑定参数，来进行指定field values或者tag vaules。

        Query语法：
        --data-urlencode 'q= SELECT [...] WHERE [ < field_key > | < tag_key > ] = $< placeholder_key >'

        Map语法：
        --data-urlencode 'params={"< placeholder_key >":[ < placeholder_float_field_value > | < placeholder_integer_field_value > | "< placeholder_string_field_value >" | < placeholder_boolean_field_value > | "< placeholder_tag_value >" ]}'

示例1：发送多条Query命令

    $ curl -G 'http://localhost:8086/query?db=mydb&epoch=s' --data-urlencode 'q=SELECT * FROM "mymeas";SELECT mean("myfield") FROM "mymeas"'

    {"results":[{"series":[{"name":"mymeas","columns":["time","myfield","mytag1","mytag2"],"values":[[1463779800,12,"1",null],[1463779820,11,"2",null],[1463779840,18,null,"1"],[1463779860,19,null,"3"]]}]},{"series":[{"name":"mymeas","columns":["time","mean"],"values":[[0,15]]}]}]}

示例2：以CSV格式返回请求结果

    curl -H "Accept: application/csv" -G 'http://localhost:8086/query?db=mydb' --data-urlencode 'q=SELECT * FROM "mymeas" LIMIT 3'

    name,tags,time,tag1,tag2,value
    mymeas,,1478030187213306198,blue,tag2,23
    mymeas,,1478030189872408710,blue,tag2,44
    mymeas,,1478030203683809554,blue,yellow,101

示例3：通过文件的形式导入Queries

    curl -F "q=@queries.txt" -F "async=true" 'http://localhost:8086/query'
    文本内容如下:
    CREATE DATABASE mydb;
    CREATE RETENTION POLICY four_weeks ON mydb DURATION 4w REPLICATION 1;

示例4：通过WHERE字段指定tag value

    curl -G 'http://localhost:8086/query?db=mydb' --data-urlencode 'q=SELECT * FROM "mymeas" WHERE "mytagkey" = $tag_value' --data-urlencode 'params={"tag_value":"mytagvalue1"}'

    {"results":[{"series":[{"name":"mymeas","columns":["time","myfieldkey","mytagkey"],"values":[["2016-09-05T18:25:08.479629934Z",9,"mytagvalue1"],["2016-09-05T18:25:20.892472038Z",8,"mytagvalue1"],["2016-09-05T18:25:30.408555195Z",10,"mytagvalue1"],["2016-09-05T18:25:39.108978991Z",111,"mytagvalue1"]]}]}]}

示例5：通过WHERE字段指定数字区间

    curl -G 'http://localhost:8086/query?db=mydb' --data-urlencode 'q=SELECT * FROM "mymeas" WHERE "myfieldkey" > $field_value' --data-urlencode 'params={"field_value":9}'

    {"results":[{"series":[{"name":"mymeas","columns":["time","myfieldkey","mytagkey"],"values":[["2016-09-05T18:25:30.408555195Z",10,"mytagvalue1"],["2016-09-05T18:25:39.108978991Z",111,"mytagvalue1"],["2016-09-05T18:25:46.587728107Z",111,"mytagvalue2"]]}]}]}

示例6：通过WHERE字段指定多个条件

    curl -G 'http://localhost:8086/query?db=mydb' --data-urlencode 'q=SELECT * FROM "mymeas" WHERE "mytagkey" = $tag_value AND  "myfieldkey" > $field_value' --data-urlencode 'params={"tag_value":"mytagvalue2","field_value":9}'

    {"results":[{"series":[{"name":"mymeas","columns":["time","myfieldkey","mytagkey"],"values":[["2016-09-05T18:25:46.587728107Z",111,"mytagvalue2"]]}]}]}

### /write

/wirte只支持POST的HTTP请求，使用该Endpoint可以写数据到已存在的数据库中。\
**定义：**\
POST <http://localhost:8086/write>

**Query参数说明：**

  参数                                 是否可选   描述
  ------------------------------------ ---------- --------------------------------------------------------------------------------------------------------------------------------------------
  consistency=\[any,one,quorum,all\]   可选       设置point的写入一致性，默认是one.详细的请参考[**HERE**](https://docs.influxdata.com/enterprise/v1.1/concepts/clustering#write-consistency)
  db={db\_name}                        必选       设置数据库名
  precision=\[h,m,s,ms,u,n\]           可选       指定时间戳的精度，默认是ns
  p={password}                         可选       如果设置了认证，则需要用户密码
  rp={rp\_name}                        可选       设置查询的rp。如果没有设置，则查询默认的rp
  u={username}                         可选       如果设置了认证，则需要用户密码

示例1：使用秒级的时间戳，将一个point写入数据库mydb

    $ curl -i -XPOST "http://localhost:8086/write?db=mydb&precision=s" --data-binary 'mymeas,mytag=1 myfield=90 1463683075'

示例2：将一个point写入数据库mydb，并指定RP为myrp

    $ curl -i -XPOST "http://localhost:8086/write?db=mydb&rp=myrp" --data-binary 'mymeas,mytag=1 myfield=90'

示例3：使用HTTP认证的方式，将一个point写入数据库mydb

    $ curl -i -XPOST "http://localhost:8086/write?db=mydb&u=myusername&p=mypassword" --data-binary 'mymeas,mytag=1 myfield=91'

示例4：使用基础认证的方式，将一个point写入数据库mydb

    $ curl -i -XPOST -u myusername:mypassword "http://localhost:8086/write?db=mydb" --data-binary 'mymeas,mytag=1 myfield=91'

**数据请求体：**

    --data-binary '< Data in Line Protocol format >'

所有写入的数据必须是二进制，且使用[**Line
Protocol**](https://docs.influxdata.com/influxdb/v1.1/concepts/glossary/#line-protocol)格式。

示例1：写多个points到数据库中,需要使用新的一行

    $ curl -i -XPOST "http://localhost:8086/write?db=mydb" --data-binary 'mymeas,mytag=3 myfield=89
    mymeas,mytag=2 myfield=34 1463689152000000000'

示例2：通过导入文件的形式，写入多个points。需要使用@来指定文件

    $ curl -i -XPOST "http://localhost:8086/write?db=mydb" --data-binary @data.txt
    文件内容如下
    mymeas,mytag1=1 value=21 1463689680000000000
    mymeas,mytag1=1 value=34 1463689690000000000
    mymeas,mytag2=8 value=78 1463689700000000000
    mymeas,mytag3=9 value=89 1463689710000000000

**响应的状态码：**

  HTTP状态码                  描述
  --------------------------- ---------------------------------------------------------------------------------
  204 No Content              成功
  400 Bad Request             不能接受的请求。可能是Line Protocol语法错误；写入错误的field values类型；等。。
  404 Not Fount               不能接受的请求。可能是数据库不存在，或者别的原因
  500 Internal Server Error   系统超负荷了或者明显受损。可能是用户企图去写一个不存在的RP。或者别的原因

InfluxDB集群化
--------------

InfluxDB v0.12及以上版本已经不再开源其集群部分代码，转为商业版本功能。\
可以参考支持集群的最新版本v0.11。

参考资料
--------

1.官方概念介绍：
[https://docs.influxdata.com/i...](https://docs.influxdata.com/influxdb/v1.1/concepts/key_concepts/)\
2.InfluxDB详解之TSM存储引擎解析(一)：
[http://blog.fatedier.com/2016...](http://blog.fatedier.com/2016/08/05/detailed-in-influxdb-tsm-storage-engine-one/)\
　InfuxDB详解之TSM存储引擎解析(二)：[http://blog.fatedier.com/2016...](http://blog.fatedier.com/2016/08/15/detailed-in-influxdb-tsm-storage-engine-two/)

