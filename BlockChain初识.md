---
title: BlockChain初识
tags: BlockChain,区块链,Hyperledger,fabric
grammar_cjkRuby: true
---
## 区块链介绍

<hr /> 

区块链最早是在比特币项目中，为比特币的运行提供一个分布式的记账平台。而区块链技术发展到现在，对于区块链的定义为，一个区块链是一个分布式的数据库，该数据库维持一个持续增长的数据记录链，并且能够防止数据被篡改。它由数据结构块组成，该结构块在初始的区块链实现中持有专有的数据，并且数据和程序都保存到一些最近的实现里面，每一个区块都保存一些个人交易数据和区块的执行结果。每一个区块都包含一个时间戳和上一个区块的信息。

区块链是一种去中心化的记录技术。也就是说，参与到系统上的任何一个节点，可能不属于同一组织、彼此无需信任；区块链数据由所有节点功能维护，每个参与维护节点都能复制获得一份完整的记录的拷贝。

### 基本原理

<hr />


区块链的基本概念有：
 - 交易(Transaction)：也就是一次操作，使得账本状态发生一次改变，例如添加一条记录。
 - 区块(Block)：记录一段时间内发生的交易和状态结果，是对当前账本状态的一次共识。
 - 链：由一个个区块按照发生顺序串联而成，是整个状态变化的日志记录。

如果把区块链作为一个状态机，则每次交易就是试图改变一次状态，而每次共识生成的区块，就是参数者对于区块中所有交易内容导致状态改变的结果进行确认。


![区块链示例][1]


  [1]: ./images/QQ%E5%9B%BE%E7%89%8720170312144942.png "QQ图片20170312144942"
  
  在实现中，首先假设存在一个分布式的数据记录本，这个记录本只允许添加，不允许删除。其结构是一个线性的链表，由一个个”区块“串联组成，这也是”区块链“的由来。新的数据要加入，必须放到一个新的区块中。而这个区块（以及块里的交易）是否合法，可以通过一些手段快速检验出来。维护节点可以提议一个新的区块，然而必须经过一定的共识机制来对最终选择的区块达成一致。
  
  以比特币为例来看如何使用区块链技术，客户端发起一项交易后，会广播到网络中并等待确认。网络中的节点会将一些等待确认的加以记录打包在一起（此外还要包括此前区块链的哈希值等信息），组成一个候选区块。然后，试图找到一个nonce串放到区块里，使得候选区块的hash结构满足一定条件（比如，小于某个值）。一旦算出这个区块在各式上合法，则进行全网广播。大家拿到提案区块，进行验证，发现确实符合约定条件了，就承认这个区块是一个合法的新区块，被添加到链上。
  
  比特币的这种基于算力的共识机制被称为Proof of Work(PoW)。目前，要让hash结果满足一定条件并无已知的启发式算法，只能进行暴力尝试。尝试的次数越多，算出来的概率越大。通过调节对hash结果的限制，比特币网络控制约10分钟平均算出来一个合法的区块。算出来的节点将得到区块中所有交易的管理费和协议固定发放的奖励费。(目前是12.5比特币，每4年减半)，也就是挖矿。
  
  ### 分类
  
  <hr />
  
根据参与者不同，区块链可以分为公开(Public)链，联盟(Consortium)链和私有(Private)链。
  
公共链，顾名思义，任何人都可以参与使用和维护，典型的如比特币区块链，信息是完全公开的。

如果引入许可机制，包括私有链和联盟链两种。

私有链，则是集中管理者进行限制，只有得到内部少数人可以使用，信息不公开。

联盟链介于两者之间，由若干组织一起合作维护一条区块链，该区块链的使用必须是有权限的管理，相关信息会得到保护，如银联组织。

## Hyperledger--超级账本项目

<hr /> 

### 安装部署

在这里hyperledger的安装部署我们使用docker方案进行一键式部署，方便简单。

#### 安装Docker

首先，安装Docker。

```
	curl -fsSL https://get.docker.com/ | sh
```

重启docker服务：

```
	sudo service docker restart
```

接着，安装docker-compose

```
	sudo aptitude install python-pip

	sudo pip install docker-sompose>=1.7.0

```

下载镜像，目前我们使用最新的1.0版本进行学习。

```
	$ ARCH=x86_64
	$ BASE_VERSION=1.0.0-preview
	$ PROJECT_VERSION=1.0.0-preview
	$ IMG_VERSION=0.8.4
	$ docker pull yeasy/hyperledger-fabric-base:$IMG_VERSION \
	  && docker pull yeasy/hyperledger-fabric-peer:$IMG_VERSION \
	  && docker pull yeasy/hyperledger-fabric-orderer:$IMG_VERSION \
	  && docker pull yeasy/hyperledger-fabric-ca:$IMG_VERSION \
	  && docker pull yeasy/blockchain-explorer:latest \
	  && docker tag yeasy/hyperledger-fabric-peer:$IMG_VERSION hyperledger/fabric-peer \
	  && docker tag yeasy/hyperledger-fabric-orderer:$IMG_VERSION hyperledger/fabric-orderer \
	  && docker tag yeasy/hyperledger-fabric-ca:$IMG_VERSION hyperledger/fabric-ca \
	  && docker tag yeasy/hyperledger-fabric-base:$IMG_VERSION hyperledger/fabric-baseimage \
	  && docker tag yeasy/hyperledger-fabric-base:$IMG_VERSION hyperledger/fabric-ccenv:$ARCH-$BASE_VERSION \
	  && docker tag yeasy/hyperledger-fabric-base:$IMG_VERSION hyperledger/fabric-baseos:$ARCH-$BASE_VERSION

```

接着启动fabric 1.0网络。

下载Compose模板文件。

```
	git clone https://github.com/yeasy/docker-compose-files
```

进入`hyperledger/1.0`目录，查看相关文件，其功能为：

 - `peers.yml`：包含peer节点的服务模板。
 - `docker-compose.yml`：启动1个最小化环境，包括1个peer节点，一个Orderer节点，1个CA节点。

通过下面命令快速启动：

```
docker-sompose up
```

注意输出日志中无错误信息。

此时，系统中包括三个容器：

```

	$ docker ps
	CONTAINER ID        IMAGE                                                                                                                                                  COMMAND                  CREATED             STATUS              PORTS                                             NAMES
	2367ccb6463d        hyperledger/fabric-peer          "peer node start"        15 minutes ago      Up 15 minutes       7050/tcp, 7052-7059/tcp, 0.0.0.0:7051->7051/tcp   fabric-peer0
	02eaf86496ca        hyperledger/fabric-orderer       "orderer"                15 minutes ago      Up 15 minutes       0.0.0.0:7050->7050/tcp                            fabric-orderer
	71c2246e1165        hyperledger/fabric-ca            "fabric-ca server sta"   15 minutes ago      Up 15 minutes       7054/tcp, 0.0.0.0:8888->8888/tcp                  fabric-ca

```

#### 测试chainnode操作

启动fabric网络后，可以进行chainnode操作，验证网络启动正常。

*部署chainnode*




