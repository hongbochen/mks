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

