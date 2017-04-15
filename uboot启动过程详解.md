---
title:uboot启动过程详解
tags: uboot,启动验证
grammar_cjkRuby: true
---
在android启动过程中，首先启动的便是uboot，uboot是负责引导内核装入内存启动或者是引导recovery模式的启动。现在在很多android的uboot的启动过程中，都需要对内核镜像和ramdisk进行验证，来保证android系统的安全性，如果在uboot引导过程中，如果内核镜像或ramdisk刷入的是第三方的未经过签名认证的相关镜像，则系统无法启动。