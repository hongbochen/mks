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

为了横向扩展，你可以设置一个Kaa集群的每一个几点都是操作服务使能的。在这个情况下，所有的操作服务的实例当前都是在运行的。。

For the purpose of horizontal scaling, you can set up a Kaa cluster with Operations service enabled for every node. In this case, all instances of Operations service will function concurrently. In case of an Operations service outage, previously connected endpoints switch to other available Operations services automatically. Kaa server can re-balance the load at run time, thus effectively routing endpoints to the less loaded nodes in the cluster.