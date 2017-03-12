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

#### 测试chaincode操作

启动fabric网络后，可以进行chaincode操作，验证网络启动正常。

*部署chaincode*

通过日下命令进入容器peer0。

```
	docker exec -it fabric-peer0 bash

```

在容器中执行部署命令install和instantiate，注意输出日志无错误提示，最终返回的结果应该为`response:<status:200 message:"OK" payload:"100" >`。

其中，peer默认加入到了名为testchainid的channel中，并在此channel中执行instantiate/invoke/query命令，详细的解释壳通过`peer --help`查看。

```
	root@peer0:/go/src/github.com/hyperledger/fabric# peer chaincode install -n test_cc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a","100","b","200"]}' -v v0
	...
	[container] WriteGopathSrc -> INFO 001 rootDirectory = /go/src
	[container] WriteFolderToTarPackage -> INFO 002 rootDirectory = /go/src
	Installed remotely response:<status:200 payload:"OK" > 
	[main] main -> INFO 003 Exiting.....

```

之后执行`peer chaincode instantiate`命令。

```
	root@peer0:/go/src/github.com/hyperledger/fabric# peer chaincode instantiate -n test_cc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a","100","b","200"]}' -v v0
	...
	[chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
	[chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
	[main] main -> INFO 003 Exiting.....

```

此时，系统中生成类型`dev-peer0-test_cc-v0`的chaincode Docker镜像，和相同名称的容器。

```
	$ docker ps
	CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                             NAMES
	edc9740c265c        dev-peer0-test_cc-v0        "/opt/gopath/bin/t..."   34 minutes ago      Up 34 minutes                                                         dev-peer0-test_cc-1.0
	2367ccb6463d        hyperledger/fabric-peer      "peer node start"        36 minutes ago      Up 36 minutes       7050/tcp, 7052-7059/tcp, 0.0.0.0:7051->7051/tcp   fabric-peer0
	02eaf86496ca        hyperledger/fabric-orderer   "orderer"                36 minutes ago      Up 36 minutes       0.0.0.0:7050->7050/tcp                            fabric-orderer
	71c2246e1165        hyperledger/fabric-ca        "fabric-ca server ..."   36 minutes ago      Up 36 minutes       7054/tcp, 0.0.0.0:8888->8888/tcp 

```

*查询chaincode*

对部署成功的chaincode执行查询操作，查询`a`的余额。

同样，在peer0容器中执行如下命令，注意输出无错误信息，最后的结果为`Query Result:100`。

```
	root@peer0:/go/src/github.com/hyperledger/fabric# peer chaincode query  -n test_cc  -c '{"Args":["query","a"]}'
	Query Result: 100
	2017-02-20 07:12:10.020 UTC [main] main -> INFO 001 Exiting.....
```

或者是也可以使用下列方式查询，最后的结果为`<status:200 message:"OK" payload:"100" >`。

```
	root@peer0:/go/src/github.com/hyperledger/fabric# peer chaincode invoke -n test_cc -c '{"Args":["query","a"]}'
	[chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Invoke result: version:1 response:<status:200 message:"OK" payload:"100" > payload:"\n \n3\342Nbxl\340\332\367\220fT}]\371\027Q\246A\332nI\242&#5i\230\220Bx\0224\n(\002\004lccc\001\007test_cc\004\001\001\001\001\000\000\007test_cc\001\001a\004\001\001\001\001\000\000\032\010\010\310\001\032\003100" endorsement:<endorser:"\n\007DEFAULT\022\232\007-----BEGIN -----\nMIICjDCCAjKgAwIBAgIUBEVwsSx0TmqdbzNwleNBBzoIT0wwCgYIKoZIzj0EAwIw\nfzELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNh\nbiBGcmFuY2lzY28xHzAdBgNVBAoTFkludGVybmV0IFdpZGdldHMsIEluYy4xDDAK\nBgNVBAsTA1dXVzEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMTYxMTExMTcwNzAw\nWhcNMTcxMTExMTcwNzAwWjBjMQswCQYDVQQGEwJVUzEXMBUGA1UECBMOTm9ydGgg\nQ2Fyb2xpbmExEDAOBgNVBAcTB1JhbGVpZ2gxGzAZBgNVBAoTEkh5cGVybGVkZ2Vy\nIEZhYnJpYzEMMAoGA1UECxMDQ09QMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE\nHBuKsAO43hs4JGpFfiGMkB/xsILTsOvmN2WmwpsPHZNL6w8HWe3xCPQtdG/XJJvZ\n+C756KEsUBM3yw5PTfku8qOBpzCBpDAOBgNVHQ8BAf8EBAMCBaAwHQYDVR0lBBYw\nFAYIKwYBBQUHAwEGCCsGAQUFBwMCMAwGA1UdEwEB/wQCMAAwHQYDVR0OBBYEFOFC\ndcUZ4es3ltiCgAVDoyLfVpPIMB8GA1UdIwQYMBaAFBdnQj2qnoI/xMUdn1vDmdG1\nnEgQMCUGA1UdEQQeMByCCm15aG9zdC5jb22CDnd3dy5teWhvc3QuY29tMAoGCCqG\nSM49BAMCA0gAMEUCIDf9Hbl4xn3z4EwNKmilM9lX2Fq4jWpAaRVB97OmVEeyAiEA\n25aDPQHGGq2AvhKT0wvt08cX1GTGCIbfmuLpMwKQj38=\n-----END -----\n" signature:"0D\002 IC\266\236\222E\370\243\221\272\312k\007\336\306\265\034_\tT\321O@\247\241\267\334\315\311\231\264E\002 \010\375/\220\232h\322IP\350B\222@\200\201\204\20140BI\261\334\211\023\305F\345\001\260\250." > 
	[main] main -> INFO 002 Exiting.....
	
```

类似的，查询`b`的余额，注意最终返回结果为`Query Result: 200`。

```
	root@peer0:/go/src/github.com/hyperledger/fabric# peer chaincode query  -n test_cc  -c '{"Args":["query","b"]}'
	Query Result: 200
	[main] main -> INFO 001 Exiting.....
```

或者为

```
	root@peer0:/go/src/github.com/hyperledger/fabric# peer chaincode invoke -n test_cc -c '{"Args":["query","b"]}'
	[chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Invoke result: version:1 response:<status:200 message:"OK" payload:"200" > payload:"\n \366\340\355\3350\202\326\213\367p\222\364r\326\212\177\240\214\204\254\364\232\312\227\242(Z9\010a\342\241\0224\n(\002\004lccc\001\007test_cc\004\001\001\001\001\000\000\007test_cc\001\001b\004\001\001\001\001\000\000\032\010\010\310\001\032\003200" endorsement:<endorser:"\n\007DEFAULT\022\232\007-----BEGIN -----\nMIICjDCCAjKgAwIBAgIUBEVwsSx0TmqdbzNwleNBBzoIT0wwCgYIKoZIzj0EAwIw\nfzELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNh\nbiBGcmFuY2lzY28xHzAdBgNVBAoTFkludGVybmV0IFdpZGdldHMsIEluYy4xDDAK\nBgNVBAsTA1dXVzEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMTYxMTExMTcwNzAw\nWhcNMTcxMTExMTcwNzAwWjBjMQswCQYDVQQGEwJVUzEXMBUGA1UECBMOTm9ydGgg\nQ2Fyb2xpbmExEDAOBgNVBAcTB1JhbGVpZ2gxGzAZBgNVBAoTEkh5cGVybGVkZ2Vy\nIEZhYnJpYzEMMAoGA1UECxMDQ09QMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE\nHBuKsAO43hs4JGpFfiGMkB/xsILTsOvmN2WmwpsPHZNL6w8HWe3xCPQtdG/XJJvZ\n+C756KEsUBM3yw5PTfku8qOBpzCBpDAOBgNVHQ8BAf8EBAMCBaAwHQYDVR0lBBYw\nFAYIKwYBBQUHAwEGCCsGAQUFBwMCMAwGA1UdEwEB/wQCMAAwHQYDVR0OBBYEFOFC\ndcUZ4es3ltiCgAVDoyLfVpPIMB8GA1UdIwQYMBaAFBdnQj2qnoI/xMUdn1vDmdG1\nnEgQMCUGA1UdEQQeMByCCm15aG9zdC5jb22CDnd3dy5teWhvc3QuY29tMAoGCCqG\nSM49BAMCA0gAMEUCIDf9Hbl4xn3z4EwNKmilM9lX2Fq4jWpAaRVB97OmVEeyAiEA\n25aDPQHGGq2AvhKT0wvt08cX1GTGCIbfmuLpMwKQj38=\n-----END -----\n" signature:"0E\002!\000\335\t\234\347\367&\316-G~J\336u\tn\035\030U\314\021\227Z\241U\307+\\^>\230\216k\002 qk;\276\007\312'\376\022\267\342h\2620>\317\353\232\\\223\334U\372xu\2275\274\327\345fH" >
	[main] main -> INFO 002 Exiting.....
```




