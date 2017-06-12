---
title: Kaa IoT平台学习（二） 
tags: Kaa Cluster, Kaa Instance
---

在这片文章中，主要讨论在Kaa架构和逻辑设计下的功能性概念。

Kaa IoT平台由Kaa server，Kaa扩展和端点SDKs组成。

 - kaa服务器是平台的后端部分。他被用于去管理租户，应用，用户和设备。Kaa服务器暴露了集成接口并且提供了管理能力。
 - kaa扩展是独立的软件模块，他提升了平台的功能性。
 - 端点SDK是为多种多样的Kaa平台特征提供客户端的API并且处理通信，数据编集，持久性等的一个库。Kaa SDK被设计区促进客户端应用的创造性来运行在各种各样连接的设备上。然而，不使用Kaa端点SDK的客户端应用也是有可能的。以不同的编程语言有多种不同的端点SDK。

### Kaa集群

Kaa服务器节点使用Apache的ZooKeeper来与服务合作。互相连接的节点和特别的Kaa实例组成了一个Kaa集群。Kaa集群需要Nosql和SQL数据库实例来保存端点数据和原语。

![enter description here][1]


  [1]: ./images/high-level-architecture.png "high-level-architecture"
  
 位于集群中的Kaa节点运行了Control,Operation和Bootstrap服务的组合。

#### Control服务

Kaa控制服务管理所有的系统数据，处理来自Web UI和外部集成系统的API请求，并且向Operaion服务发送通知。控制服务通过持续的接收来自ZooKeeper的信息来维持一个最新的可操作服务列表。除此之外，控制服务运行嵌入的使用控制服务API的管理web UI组件，来想用户提供方便的基于web的接口来管理租户，用户账户，应用数据等。

为了支持高可用性，一个Kaa集群至少有两个节点是使能控制服务的。在高可用性模式中，其中的一个控制服务是活动的，另外一个是待机模式。一旦活动的控制服务失效了，ZooKeeper会唤醒其中一个待机控制服务并且将它升级成为活动控制服务。

#### 操作服务

操作服务最基础的角色就是与当前多个端点进行通信。操作服务处理端点请求并且把数据发送给他们。

为了横向扩展，你可以设置一个Kaa集群的每一个几点都是操作服务使能的。在这个情况下，所有的操作服务的实例当前都是在运行的。如果一个操作服务意外终止了，之前连接端点自动转换到其他可用的操作服务中去。Kaa服务器在运行时可以重新负载均衡，所以在集群中路由端点到低负载的节点中的效率是非常高的。

#### 引导程序服务

Kaa Bootstrap服务发送关于操作服务连接参数的信息到端点中。取决于配置的协议栈，连接参数可能包括IP地址，TCP端口，安全证书等。Kaa SDK包含一个在集群中预生成的Bootstrao可用列表，他被用于生成SDK库。在这个列表中的端点查询Bootstrap服务为当前可用操作服务取回连接。Bootstrap服务通过和ZooKeeper服务合作来维持他们的可用操作服务的列表。

### 第三方组件

#### Zookeeper

Apache ZooKeeper使能在Kaa集群节点之间高可靠性分布式合作。每一个Kaa节点持续的推送关于连接参数，使能的服务和回应的服务负载的信息，其他Kaa节点使用这个信息去获取他们兄弟的列表并且与他们进行通信。活动的控制服务在SDK生成期间使用关于可用Bootstrap服务和他们连接参数的信息。

#### SQL database

SQL数据库实例被用于存储租户，应用，端点组合其他原语，他们不随着端点的增加而增长。


一个Kaa集群的高可用性通过在HA模式下部署SQL数据库被实现了。Kaa现在官方支持MariaDB和PostgreSQL最为嵌入的SQL数据库。

#### NoSQL database

NoSql数据库实例被用于存储端点关系数据，这些数据随着端点的增加成线性增长。

NoSQL数据库节点可以和Kaa节点一样放在相同的为或虚拟机上，并且为了这个系统的高可用性，他应该在HA模式下被部署。Kaa官方支持Apache Cassandra和MongoDB作为嵌入的NoSQL数据库。

#### Internode communications

Kaa服务使用Apache Thirft来进行进程和节点之间的通信。每一个服务使用ZooKeeper包含关于他的兄弟的元数据。这个元数据包含关于Thrift主机和端口的信息。
Kaa services use Apache Thirft to communicate across processes and nodes. Each service obtains metadata about its siblings using Apache ZooKeeper. This metadata contains information about the Thrift host and port.