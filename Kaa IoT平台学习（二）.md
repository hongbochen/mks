---
title: Kaa IoT平台学习（二） 
tags: Kaa Cluster, Kaa Instance
---

在这片文章中，主要讨论在Kaa架构和逻辑设计下的功能性概念。

Kaa IoT平台由Kaa server，Kaa扩展和端点SDKs组成。

 - kaa服务器是平台的后端部分。他被用于去管理租户，应用，用户和设备。Kaa服务器暴露了集成接口并且提供了管理能力。
 - kaa扩展是独立的软件模块，他提升了平台的功能性。
 - 端点SDK是为多种多样的Kaa平台特征提供客户端的API并且处理通信，数据编集，持久性等的一个库。Kaa SDK被设计区促进客户端应用的创造性来运行在各种各样连接的设备上。然而，不使用Kaa端点SDK的客户端应用也是有可能的。

Endpoint SDK is a library that provides client-side APIs for various Kaa platform features and handles communication, data marshalling, persistence, etc. Kaa SDKs are designed to facilitate the creation of client applications to be run on various connected devices. However, client applications that do not use Kaa endpoint SDK are also possible. There are several endpoint SDK types available in different programming languages.
