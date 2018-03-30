---
title: "Mac下搭建以太坊私有链"
date: 2017-09-09T10:22:26+08:00
draft: false 
description: ""
slug: "setup-ethereum-private-network-on-mac" 
tags: 
- "以太坊"
topics: 
- 编程语言与开发
---

在熟悉了解以太坊Ethereum时，为加快测试和掌握。在本机搭建私有链环境是必须的。
小编摸索一段时间，总算了解到以太坊的总体运行环境。下面以go-ethereum和以太坊钱包为例，详细步骤记录如何在Mac下搭建以太坊私有链运行环境。
## 配置前环境准备

### 安装以太坊客户端
实际以太坊运行有多个不同语言实现的客户端，是多样的。可为不同用户选择他们所熟悉的客户端。

|客户端|开发语言|开发者|
|----|----|----|
| [go-ethereum] | Go | [以太坊基金会][Ethereum Foundation]|  
| [Parity] | Rust | [Ethcore] |  
| [cpp-ethereum] | C++ | [以太坊基金会][Ethereum Foundation] | 
| [pyethapp] | Python | [以太坊基金会][Ethereum Foundation] |  
| [ethereumjs-lib] | Javascript | [以太坊基金会][Ethereum Foundation] |  
| [Ethereum(J)] | Java | [ether camp] |  
| [ruby-ethereum] | Ruby | [国人Xie Jan][Jan Xie] |   
| [ethereumH] | Haskell | [BlockApps] |  
因为以太坊客户端以 go-ethereum 为主，故搭建私有链也使用该客户端。

+ [Mac OS X上安装说明](https://github.com/ethereum/go-ethereum/wiki/Installation-Instructions-for-Mac)
+ [Windows 上安装说明](https://github.com/ethereum/go-ethereum/wiki/Installation-instructions-for-Windows)

在Mac上安装细节如下：
在使用brew前，需自行安装brew软件管理工具(http://brew.sh)。
因为以太坊客户端涉及到多个依赖安装，故先tap再安装
```shell 
brew tap ethereum/ethereum 
brew install ethereum 
``` 
可通过参数`--devel`直接安装开发版本(**可选**)
```shell 
brew install ethereum --devel 
``` 
安装完成后，可执行命令`geth version`查看版本信息，结果如下：
```text
Geth
Version: 1.6.7-stable
Git Commit: ab5646c532292b51e319f290afccf6a44f874372
Architecture: amd64
Protocol Versions: [63 62]
Network Id: 1
Go Version: go1.9
Operating System: darwin
GOPATH=/Users/one/Documents/dev/go
GOROOT=/usr/local/Cellar/go/1.9/libexec
```
其中 `geth`为客户端命令。


###  初始化创世区块
因为geth默认的文件存储路径是：`$HOME/Library/Ethereum/` ，故为运行方便，小编将配置文件等均放置在此目录。

>**坑** : geth 是支持指定存储目录的`--datadir`，但后面会遇到各种奇怪问题，如矿工一直不挖矿[Issue][MinerIssue]等。

```json
{
  "nonce": "0x0000000000000042",
  "difficulty": "0x020000",
  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "timestamp": "0x00",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "extraData": "0x11bbe8db4e347b4e8c937c1c8370e4b5ed33adb3db69cbdb7a38e1e50b1b82fa",
  "gasLimit": "0x4c4b40",
  "config": {
      "chainId": 15,
      "homesteadBlock": 0,
      "eip155Block": 0,
      "eip158Block": 0
  },
  "alloc": { }
}
```
将创世初始化文件保存到`$HOME/Library/Ethereum/`下文件名genesis.json。
初始区块是区块链的起始 — 第一个区块，区块0，唯一没有指向前面区块的一个区块。协议确保其他节点不会和你的区块链一致，除非他们和你有相同的初始区块，这样你想创建多少私有测试网区块链，就可以创建多少！

执行初始化：
```shell
cd $HOME/Library/Ethereum/
geth  init genesis.json
```
执行后，将提示成功初始化创世区。
```text
geth  init genesis.json
WARN [09-09|10:27:44] No etherbase set and no accounts found as default
INFO [09-09|10:27:44] Allocated cache and file handles        database=/Users/one/Library/Ethereum/geth/chaindata cache=16 handles=16
INFO [09-09|10:27:44] Writing custom genesis block
INFO [09-09|10:27:44] Successfully wrote genesis state        database=chaindata                                        hash=bd0e7a…9aadd0
INFO [09-09|10:27:44] Allocated cache and file handles        database=/Users/one/Library/Ethereum/geth/lightchaindata cache=16 handles=16
INFO [09-09|10:27:44] Writing custom genesis block
INFO [09-09|10:27:44] Successfully wrote genesis state        database=lightchaindata                                        hash=bd0e7a…9aadd0
``` 
此时可看到该目录下会多出两个文件夹:geth和keystore
```text
├── genesis.json
├── geth
│  ├── chaindata
│  │  ├── 000001.log
│  │  ├── CURRENT
│  │  ├── LOCK
│  │  ├── LOG
│  │  └── MANIFEST-000000
│  └── lightchaindata
│      ├── 000001.log
│      ├── CURRENT
│      ├── LOCK
│      ├── LOG
│      └── MANIFEST-000000
└── keystore
```

### 初始化账号
向将密码明文保存到文本文件中，本文中密码将全部是`abc`
```shell
cd $HOME/Library/Ethereum/
echo "abc" > pwd.txt 
```
运行命令创建账户
```
geth  -password pwd.txt account new
WARN [09-09|10:33:10] No etherbase set and no accounts found as default
Address: {e7a614776754b7c7ef3a1ef6430d29e90411fd75}
``` 
虽然此账户可以在以太坊的JavaScript控制台中生成，但我一般直接通过命令工具生成，关于`geth account`命令还有更多信息：
```text
SAGE:
  geth account [options] command [command options] [arguments...] 

COMMANDS:
  list    汇总打印所有账号
  new     创建一个新账号
  update  更新已存在的账户
  import  导入私钥生成新的账号
  help, h 帮助
``` 
```shell
geth account list 
```
你应该可以看到上面所创建的账号。
```text 
Account #0: {e7a614776754b7c7ef3a1ef6430d29e90411fd75} keystore:///Users/one/Library/Ethereum/keystore/UTC--2017-09-09T02-33-10.598872595Z--e7a614776754b7c7ef3a1ef6430d29e90411fd75
```
表明账号的私钥保存在keystore文件夹下。

### 进入控制台
```shell
geth  --networkid 9999 console
```
运行命令，将进入控制台，打印出如下信息。该信息非常重要。能告诉你运行时环境与配置信息。

+ Allocated cache and file handles 缓存存放目录
+ Disk storage enabled for ethash caches 数据存放目录
+ Disk storage enabled for ethash DAGs DAG数据存放目录
+ IPC endpoint opened IPC地址，**重要**，关系到后续以太坊钱包的链接

```text
INFO [09-09|10:40:36] Starting peer-to-peer node              instance=Geth/v1.6.7-stable-ab5646c5/darwin-amd64/go1.9
INFO [09-09|10:40:36] Allocated cache and file handles        database=/Users/one/Library/Ethereum/geth/chaindata cache=128 handles=1024
WARN [09-09|10:40:36] Upgrading chain database to use sequential keys
INFO [09-09|10:40:36] Database conversion successful
INFO [09-09|10:40:36] Initialised chain configuration          config="{ChainID: 15 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: <nil> EIP155: 0 EIP158: 0 Metropolis: <nil> Engine: unknown}"
INFO [09-09|10:40:36] Disk storage enabled for ethash caches  dir=/Users/one/Library/Ethereum/geth/ethash count=3
INFO [09-09|10:40:36] Disk storage enabled for ethash DAGs    dir=/Users/one/.ethash                      count=2
WARN [09-09|10:40:36] Upgrading db log bloom bins
INFO [09-09|10:40:36] Bloom-bin upgrade completed              elapsed=68.449µs
INFO [09-09|10:40:36] Initialising Ethereum protocol          versions="[63 62]" network=9999
INFO [09-09|10:40:36] Loaded most recent local header          number=0 hash=bd0e7a…9aadd0 td=131072
INFO [09-09|10:40:36] Loaded most recent local full block      number=0 hash=bd0e7a…9aadd0 td=131072
INFO [09-09|10:40:36] Loaded most recent local fast block      number=0 hash=bd0e7a…9aadd0 td=131072
INFO [09-09|10:40:36] Starting P2P networking
INFO [09-09|10:40:38] UDP listener up                          self=enode://f28939fbbd6038c074f322b656f314250bbf7372523ee6d7d2fd6b67f86dba3f41cdf394ab3c57bd105cb96bec70337c6c19a09ca794b6f0c36c3d04119c7c39@[::]:30303
INFO [09-09|10:40:38] RLPx listener up                        self=enode://f28939fbbd6038c074f322b656f314250bbf7372523ee6d7d2fd6b67f86dba3f41cdf394ab3c57bd105cb96bec70337c6c19a09ca794b6f0c36c3d04119c7c39@[::]:30303
INFO [09-09|10:40:38] IPC endpoint opened: /Users/one/Library/Ethereum/geth.ipc
Welcome to the Geth JavaScript console!

instance: Geth/v1.6.7-stable-ab5646c5/darwin-amd64/go1.9
coinbase: 0xe7a614776754b7c7ef3a1ef6430d29e90411fd75
at block: 0 (Thu, 01 Jan 1970 08:00:00 CST)
datadir: /Users/one/Library/Ethereum
modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0 
```

能在控制台中执行多种命令，具体见官方文档：[JavaScript-Console Wiki][JSCWiki]。
这是一个交互式的Javascript执行环境，在这里面可以执行Javascript代码，其中>是命令提示符。在这个环境里也内置了一些用来操作以太坊的Javascript对象，可以直接使用这些对象。这些对象主要包括：

+ eth：包含一些跟操作区块链相关的方法
+ net：包含以下查看p2p网络状态的方法
+ admin：包含一些与管理节点相关的方法
+ miner：包含启动&停止挖矿的一些方法
+ personal：主要包含一些管理账户的方法
+ txpool：包含一些查看交易内存池的方法
+ web3：包含了以上对象，还包含一些单位换算的方法


## 启动挖矿 
如果你之前有部署运行过以太坊，请先将此目录下的DAG文件删除`rm -rf $HOME/.ethash/`。

参数说明
```shell
--nodiscover
```
使用这个命令可以确保你的节点不会被**非手动**添加你的人发现。否则，你的节点可能因为你与他有相同的创世文件和网络ID而被陌生人的区块链无意添加。
```shell
--maxpeers 0
```
如果你不希望其他人连接到你的测试链，可以使用maxpeers 0。反之，如果你确切知道希望多少人连接到你的节点，你也可以通过调整数字来实现。
```shell
--rpc
```
这个指令可以激活你节点上的RPC界面。它在geth中通常被默认激活。
```shell
--rpcapi "db,eth,net,web3"
```
这个命令可以决定允许什么API通过RPC进入。在默认情况下，geth可以在RPC激活web3界面。
**记住**：在RPC/IPC界面提供API，会使每个可以进入这个界面（例如dapp's）的人都有权限访问这个API。注意你激活的是哪个API。Geth会默认激活IPC界面上所有的API，以及RPC界面上的db,eth,net和web3 API。
```shell
--rpcport "8080"
```
将8000改变为你网络上开放的任何端口。Geth的默认设置是8080.
```
```shell
--rpccorsdomain "https://yushuangqi.com"
```
这个可以指示什么URL能连接到你的节点来执行RPC定制端任务。务必谨慎，输入一个特定的URL而不是wildcard ( * )，后者会使所有的URL都能连接到你的RPC实例。
```shell
--datadir "/home/TestChain1"
```
这是你的私有链数据所储存在的数据目录（在nubits下）。选择一个与你以太坊公有链文件夹分开的位置。
```shell
--identity "TestnetMainNode"
```
这会为你的节点设置一个身份，使之更容易在端点列表中被辨认出来。这个例子说明了这些身份如何在网络上出现。
```shell
--networkid 1999
```
数字，区分与其他的网络ID，以太坊公链的网络ID=1。必须区分，以放置钱包等误认为是以太坊公链。 ,2=Morden (disused), 3=Ropsten, 4=Rinkeby，默认为1。
```shell
--port 30303
```
P2P网络监听端口，默认30303。
```shell
--fast 
```
这个命令是 Geth1.6.0之前的，只会被改成`--syncmode=fast`，但该命令继续有效。配置此命令能够快速的同步区块 
```shell
--cache=1024
```
程序内置的可用内存，单位MB。默认是16MB(最小值)。可以根据服务器能力配置到56, 512, 1024 (1GB), or 2048 (2GB)。

```shell
geth --identity "OneTestETH" --rpccorsdomain "*" --nodiscover --rpcapi "*"  --fast --cache=1024 --networkid 1999  console
```
进入控制台后，可以启动矿工开始挖矿。
```shell
> miner.start(1)
```
这里的`1`表示只使用一个线程运行，如果配置过高我的MAC会卡。
```text
INFO [09-09|11:17:36] Updated mining threads                  threads=1
INFO [09-09|11:17:36] Transaction pool price threshold updated price=18000000000
INFO [09-09|11:17:36] Starting mining operation
null
> INFO [09-09|11:17:36] Commit new mining work                  number=1 txs=0 uncles=0 elapsed=134.574µs
INFO [09-09|11:17:38] Generating DAG in progress              epoch=0 percentage=0 elapsed=1.051s
INFO [09-09|11:17:39] Generating DAG in progress              epoch=0 percentage=1 elapsed=2.087s
INFO [09-09|11:17:40] Generating DAG in progress              epoch=0 percentage=2 elapsed=3.129s
INFO [09-09|11:17:41] Generating DAG in progress              epoch=0 percentage=3 elapsed=4.229s
......
INFO [09-09|11:19:24] Generating DAG in progress              epoch=0 percentage=98 elapsed=1m47.287s
INFO [09-09|11:19:25] Generating DAG in progress              epoch=0 percentage=99 elapsed=1m48.790s
INFO [09-09|11:19:25] Generated ethash verification cache      epoch=0 elapsed=1m48.792s
INFO [09-09|11:19:29] Generating DAG in progress              epoch=1 percentage=0  elapsed=1.365s
INFO [09-09|11:19:30] Generating DAG in progress              epoch=1 percentage=1  elapsed=2.666s
INFO [09-09|11:19:31] Successfully sealed new block            number=1 hash=19b30c…c712b6
INFO [09-09|11:19:31] 🔨 mined potential block                  number=1 hash=19b30c…c712b6
INFO [09-09|11:19:31] Commit new mining work                  number=2 txs=0 uncles=0 elapsed=421.087µs

```
第一次运行时将开始创建DAG文件，只需等待进度条到100，则将开始挖矿。 实际你看到的挖矿速度很快，这是因为我们已经在初始化创世区块时配置为:`"nonce": "0x0000000000000042"`。
"0x42"难度能让你在私有测试网链上快速挖以太币。几分钟就会有上百个以太币，远远超过了在网络上测试交易所需的数量。

```text
coinbase: 0xe7a614776754b7c7ef3a1ef6430d29e90411fd75
```
挖矿时必然有矿工账户，而系统默认使用创建的第一个账号。

```shell
> miner.stop()
```
停止挖矿，此时可以查看将挖出不少以太币。在控制台中可查询矿工余额。
```shell
> eth.accounts
```
这会返回到你拥有的账户地址排列。
```shell
> eth.getBalance(eth.accounts[0])
```
这个控制台指令会返回到你第一个以太坊地址。因为我们只创建了一个账号，也将是矿工的账号。
而` eth.getBalance()`返回的余额是以太币的最小面额wei，

```shell
> primary = eth.accounts[0]
> balance = web3.fromWei(eth.getBalance(primary), "ether");
```
将返回矿工的以太币余额，将wei转换为以太币ether。


## 安装使用以太坊钱包
以太坊钱包，当前以改名为Mist，到[以太坊官网][ethereum org]下载最新版本。
![](https://static.yushuangqi.com/blog/2017/42295563.png)
下载后，启动Mist,稍等片刻将在右上角显示`PRIVATE-NEW`，随后点击[Launch Application]进入主页。
![](https://static.yushuangqi.com/blog/2017/42012731.png)
主页左下角红色标记为私有链，中间账号显示的是前面步骤中创建的账号，并有注明为主账号。

![](https://static.yushuangqi.com/blog/2017/42169193.png)
此时你可以继续使用命令新建账号，也可在钱包中创建账号。以太坊钱包Mist使用教程请另行[查看][mist wiki]。

## 结尾
不知是否有疑问以太坊钱包Mist是如何关联上你运行的私有链的？

这是因为在运行私有链控制台时，实际以开启了IPC服务，显示的路径为：
```text
INFO [09-09|10:40:38] IPC endpoint opened: /Users/one/Library/Ethereum/geth.ipc
```
而钱包在启动时默认在本机查找的IPC路径，而Mac OS上默认查找路径为: `$HOME/Library/Ethereum/geth.ipc`。所以钱包启动后能自动识别到你的私有链。
其他OS的默认查找路径如下,具体可查看[源代码][ipcsrc]

+ windows: .\pipe\geth.ipc
+ linux: /.ethereum/geth.ipc
+ freebsd: /.ethereum/geth.ipc
+ sunos: /.ethereum/geth.ipc

至此，如上为Mac下搭建运行以太坊私有链环境的操作过程，如有疑问可在下方留言。

[go-ethereum]: http://ethdocs.org/en/latest/ethereum-clients/go-ethereum/index.html#go-ethereum
[Parity]:http://ethdocs.org/en/latest/ethereum-clients/parity/index.html#parity
[cpp-ethereum]:http://ethdocs.org/en/latest/ethereum-clients/cpp-ethereum/index.html#cpp-ethereum
[pyethapp]:http://ethdocs.org/en/latest/ethereum-clients/pyethapp/index.html#pyethapp
[ethereumjs-lib]:http://ethdocs.org/en/latest/ethereum-clients/ethereumjs-lib/index.html#ethereumjs-lib
[Ethereum(J)]:http://ethdocs.org/en/latest/ethereum-clients/ethereumj/index.html#ethereum-j
[ruby-ethereum]:http://ethdocs.org/en/latest/ethereum-clients/ruby-ethereum/index.html#ruby-ethereum
[ethereumH]:http://ethdocs.org/en/latest/ethereum-clients/ethereumh/index.html#ethereumh
[Ethereum Foundation]: https://ethereum.org/foundation
[ether camp]: http://www.ether.camp
[BlockApps]: http://www.blockapps.net/
[Ethcore]: https://parity.io/
[Jan Xie]: https://github.com/janx/
[JSCWiki]: https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console
[ethereum org]:https://ethereum.org/
[mist wiki]:https://github.com/EthFans/wiki/wiki/以太坊钱包-Mist-使用教程
[ipcsrc]:https://github.com/ethereum/mist/blob/master/modules/settings.js#L248
[MinerIssue]:https://github.com/ethereum/go-ethereum/issues/2174 