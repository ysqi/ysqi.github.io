---
title: "共识算法PoW之算力与挖矿难度"
date: 2018-12-04T22:57:23+08:00
slug: "consensus-algorithms-PoW-hashrate"
description: "共识算法系列之Pow第二节-算力与挖矿难度"
mathjax: true 
type: post
tags:
- 算力
- 共识算法
- 比特币
categories: 
- 区块链 

series:
- 共识算法
---

大家好，我是[虞双齐]，这篇文章是关于区块链共识算法系列课文章。上一篇文章[《共识算法PoW之由来》]({{<relref "共识机制-Pow-01.md">}})中，我们讲解了工作量证明的基本原理，核心是采取穷举法暴力寻找出一个符合难度值的随机数。这篇文章讲解比特币算力，通过本节学习，你可以掌握算力概念并能理解算力引发的悲剧。

![](/images/content/Proof-of-Work.png "需要不断的更换Nonce使得哈希值符号要求")

 

## 算力
比特币挖矿形同猜数字谜，矿工要找出一个随机数（Nonce）参与哈希运算[^1]$Hash(Block+Nonce)$，使得区块哈希值符合难度要求。
**算力**则指计算机每秒可执行哈希运算的次数，也称为哈希率（**hashrate**）。

![比特币算力](/images/content/20181028155558.jpg)

上图是当前比特币算力图表[^2]。到2017年时，比特币挖矿所需算力疯涨。这与不断研发出的新型矿机投入市场有关——这些矿机利用新的技术，拥有更强的运算能力，即单位成本下的算力在快速增长，由此带来了整体算力的提升。


### 算力单位
算力每隔千位划为一个单位，最小单位 `H=1次`，其他分部是：

+ 1 H/s = 每秒可执行一次哈希运算。
+ 1 KH/s =每秒1,000哈希（一千次）。
+ 1 MH/s =每秒1,000,000次哈希（百万次）。
+ 1 GH/s =每秒1,000,000,000次哈希（十亿次）。
+ 1 TH/s =每秒1,000,000,000,000次哈希（万亿次）。
+ 1 PH/s =每秒1,000,000,000,000,000次哈希。
+ 1 EH/s =每秒1,000,000,000,000,000,000次哈希。

如果不清楚单位简称，可以查看下面国际单位的前缀表[^3]：

|Factor    | Name |    Symbol |
|----|----|----|
|$10^{24}$|    yotta|    Y
|$10^{21}$|    zetta|    Z
|$10^{18}$|    exa    |E
|$10^{15}$|    peta|    P
|$10^{12}$|    tera|    T
|$10^{9}$|giga    |G
|$10^{6}$|mega    |M
|$10^{3}$|kilo    |k
|$10^{2}$|hecto|    h
|$10^{1}$|deka    |da |

## 挖矿难度计算

### 动态调整挖矿难度 Difficulty

![](/images/content/20181129225432563.png)

上图红色线是比特币从上线到现在的难度值变化图表。可以看到难度一直在增大，难度越大意味着矿工找出那个随机数，所需要的时间也就越长。同时可以观察到难度值是周期性调整，这是因为比特币协议期望每十分钟出一个区块，2016个区块需要两周时间完成。每出2016个区块，全网节点自动根据前面2016个出块的出款时间调整难度值。如果用时超过两周，则降低难度，反之则增加难度。 
以便维持一个区块是10分钟。更多图表数据可以到[bitcoinwisdom网站](https://bitcoinwisdom.com/bitcoin/difficulty)查看。

难度值代表着寻找随机数耗时程度，耗时越长意味着计算机所需要执行哈希计算的次数越多。一台普通笔记本电脑，每秒可以执行800次哈希运算，配置中端显卡可以算2000多次。

中本聪在自己电脑上挖出创世区块（比特币区块链的第一个区块）。原本中本聪设计的是一个公平的完全去中心化的一个数字货币系统，每个人都可以使用个人电脑进行挖矿（他也预料网络会出现挖矿的服务商（矿池））。然而，有利可图时大量新算力不断加入，矿工竞争激烈，使得单个矿工的挖矿成功率几乎为零。2011年起矿池出现，大量矿工纷纷加入矿池，以稳定收入，摊薄成本。大量算力融入，使得比特币挖矿难度越来越大。数字货币挖矿业形同军事竞备，挖矿设备不断更新迭代，不再遵循摩尔定律。 专业矿机专门针对哈希算法、散热、耗能进行优化，这脱离了比特币网络节点运行在成千上万的普通计算中并公平参与挖矿的初衷。矿池的算力占据，也使得比特币风险一直存在：51%算力攻击。


### 挖矿难度计算公式

需要多少算力才能找出一个随机数，由当前区块的挖矿难度决定，难度越大所需算力越多。但挖矿难度并不在区块信息中，只在网络节点中依据规则动态计算，公式如下：
$$ 
D = \dfrac{T_1 }{T}
$$
**T** 字母是 Target 的缩写，**D** 字母是 DiFFiculty 缩写。$T_1$ 和 $T$ 均是一个256位的大数字(big number)，其中 $T_1$ 为一个非常大的常数 **$2^{256-32}-1$**。依据公式，**$T$ 越小，挖矿难度 $D$ 越大**。

依据公式，当 $T=0$ 时，$D$无穷大，标志着无法计算出结果。幸运的是， $T$ 不会为 0，最小值为 1，此时难度值最大，为 $2^{256-32}-1=2^{224}-1$。当 $T=T_1$ 时，难度值为最小值 1。

### 目标值 Target 与挖矿难度转换
为了方便人类直观估算难度，比特币协议将大数字 $T$ 压缩为一个浮点数记录在区块头中，字段为`bits`。

如果一个区块目标值是 0x1b0404cb，则转化成 Target 值为：$0\text{x}0404cb \times  256^{(0\text{x}1b-3)}$。

$T$ 使用类浮点数的一种压缩表示法[^5]进行压缩，压缩计算过程如下：

1. 将数字转换为 256 进制。
2. 如果第一位数字大于 127（0x7f），则前面添加 0。
3. 压缩结果中的第一位存放该256进制数的位数。
4. 后面三个数存放该256进制数的前三位，如果不足三位，从后补零。

例如，将数字1000压缩，先转换为256进制数：$1000=0\text{x}03 \times 256^{2-1} + 0\text{xe}8 \times 256^{1-1}$，结果为$[0\text{x}03,0\text{xe}8]$。第一个数未超过 $0\text{x}7\text{f}$ ,则不需填 0。但长度两位低于三位，在后面补零，最终表示为：$0\text{x}0203\text{e}800$。

又比如数字 **$2^{256-32}-1$**，转换为256进制为：
```text
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
```
第一位数字 0xFF 大于 0x7f，故前面添加零后，变成：
```text
00 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
```
其长度等于 28+1=29 (0x1d)，且长度超过三位，无需补零，则压缩结果为：0x1d00FFFF。因为压缩存储容量只有才4个字节，前两字节已经被长度和添加的 00 所占用，只剩下2个字节来存储数字，这样后面的26个 FF 值被丢弃。

如果我们将压缩结果 0x1d00FFFF 解压还会是原值吗? 实际上结果是：
$T = 0\text{x}00\text{FFFF}\times256^{\times(0\text{x}1b - 3)}$= 
```text
0x00000000FFFF0000000000000000000000000000000000000000000000000000
```
解压时这个数字被截断了，不再是原来的 **$2^{256-32}-1$** 。比特币的 $T_\text{1}$ 值就是这个 0x1d00FFFF ，如果区块中 bits 为 0x1d00FFFF 则说明该区块挖矿难度为最小挖矿难度 1。

实际上，专业的矿池程序会保留被截断的FF：
```text
0x00000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
```
称上面数字为矿池难度 1(pool diFFiculty 1)。因此根据公式，区块目标值为 0x1b0404cb 的挖矿难度在挖机上看到的是：

```text
D = 0x00000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF /  
      0x00000000000404CB000000000000000000000000000000000000000000000000
    = 16307.669773817162 (pdiFF)
```
称为 pdiFF。但是一些比特币客户端可能无法精确到这么大，所以不保留尾部的FF：
```text
0x00000000FFFFFF0000000000000000000000000000000000000000000000000000
```
此时，该挖矿难度为：
```text
D = 0x00000000FFFF0000000000000000000000000000000000000000000000000000 /
    0x00000000000404CB000000000000000000000000000000000000000000000000 
  = 16307.420938523983 (bdiFF)
```  
称为 bidFF。  


### 在哪可以查看当前比特币挖矿难度
你可以在一些提供服务的网站上查看图表数据，如：

+ https://bitcoinwisdom.com/bitcoin/diFFiculty
+ https://data.bitcoinity.org/bitcoin/diFFiculty/5y?t=l
+ https://btc.com/stats/diFF

下图是写此文章时，比特币[区块 546336](https://btc.com/0000000000000000000d8FF2d0a2b90e2edf0ec7830ed7d5805e4d8a5cf4bd84) 的摘要。
![](/images/content/20181028221834.jpg)

### 根据难度值如何计算算力

现在我们知道挖矿难度是如何计算的，那么为了挖出一个区块，需要执行多次哈希运算才能找到随机数，使得区块的哈希值小于目标值呢？

前面已确定 $T1= 0\text{x1d00FFFF}$，解压为 $0\text{xFFFF} \times 2^{208}$ ，对于 难度 $D$ 的目标值： $$ D = \dfrac{T_1 }{T} \implies T = \dfrac{T_1}{D} =\dfrac{0\text{xFFFF} \times 2^{208}}{D} $$

因此，挖出难度为 $D$ 的区块预计需要计算的哈希次数为：
$$\dfrac{D \times 2^{256} } {0\text{xFFFF} \times 2^{208}} = \dfrac{ D \ast 2^{48} } {0\text{xFFFF}}  $$

目前难度计算速度要求是在10分钟内找到，即在600秒内完全计算，意味着网络算力最低必须是：
$$   \dfrac{ D \ast 2^{48} } {0\text{xFFFF} \times 600}  =  \dfrac{ D \ast 2^{32} } {600}    $$
依上计算，当 $D=1$ 时，需要每秒计算7158278次哈希，即： 7.15 Mhahs/s。


### 目标值计算源代码
在调整难度时，调整的是目标值。目标值计算公式如下，但在实际计算时有些特别处理，将目标值控制在一定范围内。
```text
新目标值= 当前目标值 * 实际2016个区块出块时间 / 理论2016个区块出块时间(2周)。 
```
+ 判断是否需要更新目标值( 2016的整数倍)，如果不是则继续使用最后一个区块的目标值
+ 计算前2016个区块出块用时
+ 如果用时低于半周，则按半周计算。防止难度增加4倍以上。
+ 如果用时高于8周，则按8周计算。防止难度降低到4倍以下。
+ 用时乘以当前难度
+ 再除以2周
+ 如果超过最大难度限制，则按最大难度处理

计算过程，Go代码如下。点击[查看bticoin C++源码](https://github.com/bitcoin/bitcoin/blob/master/src/pow.cpp#L49)
```go
var (
    bigOne = big.NewInt(1)
    // 最大难度：00000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF，2^224，0x1d00FFFF
    mainPowLimit = new(big.Int).Sub(new(big.Int).Lsh(bigOne, 224), bigOne)
    powTargetTimespan = time.Hour * 24 * 14 // 两周
)
func CalculateNextWorkTarget(prev2016block, lastBlock Block) *big.Int {
    // 如果新区块(+1)不是2016的整数倍，则不需要更新，仍然是最后一个区块的 bits
    if (lastBlock.Head.Height+1)%2016 != 0 {
        return CompactToBig(lastBlock.Head.Bits)
    }
    // 计算 2016个区块出块时间
    actualTimespan := lastBlock.Head.Timestamp.Sub(prev2016block.Head.Timestamp)
    if actualTimespan < powTargetTimespan/4 {
        actualTimespan = powTargetTimespan / 4
    } else if actualTimespan > powTargetTimespan*4 {
        // 如果超过8周，则按8周计算
        actualTimespan = powTargetTimespan * 4
    }
    lastTarget := CompactToBig(lastBlock.Head.Bits)
    // 计算公式： target = lastTarget * actualTime / expectTime
    newTarget := new(big.Int).Mul(lastTarget, big.NewInt(int64(actualTimespan.Seconds())))
    newTarget.Div(newTarget, big.NewInt(int64(powTargetTimespan.Seconds())))
    //超过最多难度，则重置
    if newTarget.Cmp(mainPowLimit) > 0 {
        newTarget.Set(mainPowLimit)
    }
    return newTarget
}
```
测试代码如下，计算的是对高度为[497951+1](https://btc.com/000000000000000000139d2fc37814f5efdc637a6c8e1b202f9ee12365b01403)出块时计算的新目标值。
```go
func TestGetTarget(t *testing.T) {
    firstTime, _ := time.Parse("2006-01-02 15:04:05", "2017-11-25 03:53:16")
    lastTime, _ := time.Parse("2006-01-02 15:04:05", "2017-12-07 00:22:42")
    prevB := Block{Head: BlockHeader{Height: 497951, Bits: 0x1800d0f6, Timestamp: lastTime}}
    prev2016B := Block{Head: BlockHeader{Height: 495936, Bits: 0x1800d0f6, Timestamp: firstTime}}
    result := CalculateNextWorkTarget(prev2016B, prevB)
    bits := BigToCompact(result)
    if bits != 0x1800b0ed {
        t.Fatalf("expect 0x1800b0ed,unexpected %x", bits)
    }
}
```


[^1]: 比特币哈希算法采用的是[SHA256](https://zh.wikipedia.org/wiki/SHA-2)进行[工作量证明](https://en.bitcoin.it/wiki/Proof_of_work)。
[^2]: 在bitcoin[实时查看比特币算力](https://charts.bitcoin.com/btc/chart/hash-rate)。
[^3]: https://physics.nist.gov/cuu/Units/prefixes.html
[^4]: bitcoin网站可查看 [算力图表](https://charts.bitcoin.com/btc/chart/hash-rate)。
[^5]: [IEEE二进制浮点数算术标准（IEEE 754）](https://en.wikipedia.org/wiki/IEEE_754)


[虞双齐]: https://yushuangqi.com
[weibo]: https://weibo.com/234665601