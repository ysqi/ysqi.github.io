---
title: "理解与计算比特币难度值Difficulty"
date: 2017-12-25T07:29:55+08:00
draft: false 
description: ""
slug: "understand-bitcoin-difficulty" 
LaTeX: true 
tags:
- "Go"
- "比特币"
- "区块链"
topics: 
- 编程语言与开发
---

 
挖矿实际就是在暴力猜谜，而要猜多少次，全凭全网共识的一个难度值。只有猜出一个数字能使得区块的哈希符合难度，才算答对谜题。

那么这个猜谜游戏由于越来越多人的加入，势必会更快猜出。所以为了维持一个恒定的游戏时间(两周)，每次游戏难度均会根据上次游戏的用时而重新计算。 

游戏越来越难，如何抢在别人前面猜出呢？所以开启了抱团团战模式（矿池）加入游戏，使得解谜速度更快也更难。速度与难度总是此消彼长。
这也是为何在2014，2015年后难度值呈几何级数式增长。当然也因解谜的设备（矿机）更新换代越来越快。


## 比特币难度值Difficulty 
难度值在区块中并不记录，仅仅是为了人类直观感受解题难度而演变出的一个浮点数。公式如下：
$$ 
diffculty = \dfrac{difficulty\\_1\\_target }{ currentTarget} 
$$  

此处的 difficulty_1_target 为一个常数，非常大的一个数字。表示矿池挖矿最大难度。**目标值越小，区块生成难度越大**。

## 难度值如何存储在区块中的
在区块中存储的是Target，但是将Target经类似于[浮点数](https://en.wikipedia.org/wiki/IEEE_754)的一种压缩表示法，字段为`bits`。例如，如果区块bits记录为0x1b0404cb，那么他表示的十六进制的Target值为：
```text
0x0404cb * 2**(8*(0x1b - 3)) = 0x00000000000404CB000000000000000000000000000000000000000000000000
```
在计算时，后面3个字节`0x0404cb`作为底，前面1字节`0x1b`表示次方数。具体压缩过程如下：

+ 将数字转换为256进制数
+ 如果第一位数字大于127(0x7f),则前面添加0
+ 压缩结果中的第一位存放该256进制数的位数
+ 后面三个数存放该256进制数的前三位，如果不足三位，则后面补零

例如，将数字1000压缩，先转换为256进制数
```text
1000 = 0x03 * 256 + 0xe8 * 1
```
那么是由两个数字构成：
```text
03   e8
```
第一个数未超过0x7f,则不需填0，但长度两位低于三位，在后面补零，最终表示为：`0x0203e800` 。

有比如数字 2^(256-32)-1,转换为256进制为：
```text
ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff
```
第一位已经超过0x7f,前面添加零：
```text
00 ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff
```
现在长度为28+1=29 (0x1d),则最终压缩结果为：`0x1d00ffffff`。此时就用精度缺失，后面的26个ff 被丢弃了，因为总共才4字节，前两字节已经被长度和添加的0所占用，只剩下2个字节来存储数字。如果我们将压缩结果0x1d00ffffff解压，会是原值吗? 实际结果为：
```text
0x00ffff *256** (0x1d - 3)  = ff ff 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
```
而此数为比特币的最大Target值，此时难度最小为1。将其结果前面添加4位0，其结果为：
```text
0x00000000FFFFFF0000000000000000000000000000000000000000000000000000
```
此最小难度值1，在矿机上一般使用保留尾部的FF，则为：
```text
0x00000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
```
称上面数字为`pool difficulty 1` 矿池难度1，矿池难度简称pdiff。而比特币客户端表示如此精度，也许是困难的，所以不保留尾部的FF,则结果为：
```text
0x00000000FFFFFF0000000000000000000000000000000000000000000000000000
```
此值，在客户端上称之为bdiff。


### 如何查看当前难度值
当前难度值，可以在许多提供服务的网站上查看图表数据：

+ https://bitcoinwisdom.com/bitcoin/difficulty
+ https://data.bitcoinity.org/bitcoin/difficulty/5y?t=l
+ https://btc.com/stats/diff

### 最大难度值是多少
按公式来看，当current_target=0时diffculty无穷大，标志着难以计算。而实际上current_target不会为0，最小current_traget=1时，难度值最大，接近2^(256-32)。

### 最小难度值是多少
当current_target=difficulty_1_target 为最大值时，难度值为最小值1。

### 根据难度值如何计算网络算力network hash rate
网络算力，表示根据难度值，要计算多少次才能找到一个随机数使得区块哈希值低于目标值。

由当前目标值Target决定当前难度值 。如果当前难度为D，则根据公式：
$$ currentTarget = \dfrac{difficulty\\_1\\_target }{D} = \dfrac{  0xffff \ast 2^{208} }{D}  $$
因此，为找到一个难度为D区块，我们需计算哈希值的次数为：
$$  \dfrac{  D \ast  2 ^{256} } { 0xffff \ast 2^{208} } = \dfrac{ D \ast 2^{48} } {0xffff} = D \ast 2^{32}  $$
目前难度计算速度要求是在10分钟内找到，即在600秒内完全计算，意味着网络算力最低必须是：
$$    \dfrac{ D \ast 2^{32} } {600}    $$
依上计算，当难度值D=1时，需要每秒计算7158278次哈希，即： 7.15 Mhahs/s。挖矿的每秒算力计算单位：

+ 1 KHash/s = 1000 Hash/s
+ 1 MHash/s = 1000 KHash/s
+ 1 GHash/s = 1000 MHash/s
+ 1 THash/s = 1000 GHash/s
+ 1 PHash/s = 1000 THash/s



## 比特币区块目标值 
目标值是一个全网统一的一个256字节的数字，非常大。 最大目标值为0x1d00ffff。

比特币区块被设计为平均每10分钟生成一个新区块。那如何才能维持区块生成速度呢？ 需要动态调整区块难度，而定期自动更新目标值，则可调整难度。所以比特币设计为每隔2016个区块时全网均会自动统计过去2016个区块生成耗时，重新计算出下一个2016个区块的目标值。
```text
按10分钟一个区块生成速度，2016个区块生成时间为2016*10分钟=14天。 
```
### 目标值计算细节
目标值计算公式如下，但在实际计算时有些特别处理，将目标值控制在一定范围内。
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
    // 最大难度：00000000ffffffffffffffffffffffffffffffffffffffffffffffffffffffff，2^224，0x1d00ffff
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



