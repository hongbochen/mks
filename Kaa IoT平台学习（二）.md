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

Kaa控制服务管理所有的系统数据，处理来自Web UI和外部集成系统的API请求，并且向Operaion服务发送通知。

Kaa Control service manages overall system data, processes API calls from the web UI and external integrated systems, and sends notifications to Operations services. Control service maintains an up-to-date list of available Operations services by continuously receiving this information from ZooKeeper. Additionally, Control service runs embedded Administrative web UI component that uses Control service APIs to provide platform users with a convenient web-based interface for managing tenants, user accounts, applications, application data, etc.

To support high availability (HA), a Kaa cluster must include at least two nodes with Control service enabled. In HA mode, one of the Control services acts as active and the other(s) function in standby mode. In case of the active Control service failure, ZooKeeper notifies one of the standby Control service and promotes it to the active Control service.