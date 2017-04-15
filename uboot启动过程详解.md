---
title:uboot启动过程详解
tags: uboot,启动验证
grammar_cjkRuby: true
---
在android启动过程中，首先启动的便是uboot，uboot是负责引导内核装入内存启动或者是引导recovery模式的启动。现在在很多android的uboot的启动过程中，都需要对内核镜像和ramdisk进行验证，来保证android系统的安全性，如果在uboot引导过程中，如果内核镜像或ramdisk刷入的是第三方的未经过签名认证的相关镜像，则系统无法启动，这样便保证了android系统的安全性。

在uboot启动过程中，是从`start.S`开始的，这里详细的细节不在赘述了，该篇文章主要学习uboot对内核镜像和ramdisk镜像的验证启动过程，同时学习一下里面的优秀巧妙的编码方式。

我们从`arch/arm/lib/board.c`的函数`board_init_r`函数开始，我们来看一下该代码：