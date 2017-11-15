---
title: RabbitMQ Hello world(java)
tags: RabbitMQ, Producing, queue, Consuming
grammar_cjkRuby: true
---

### 介绍

<hr />

RabbitMQ是一个消息代理：它接收并且转发消息。你可以把他当作一个邮局：当你把想要邮寄的邮件放到邮箱中的时候，你可以确定快递员将会最终将邮件送到你的收件人那里。以此类比，RabbitMQ是一个邮箱，邮局和一个快递员。

在RabbitMQ和邮局之间最大的不同就是，它并不处理邮件，而是接收，存储并且转发二进制数据块 - 消息。

RabbitMQ和消息，在大体上，使用一些术语。

Producing（生产）无非就是发送。发送消息的程序就是`producer（生产者）`。

![Producer][1]

一个（queue）队列是位于RabbitMQ内部的邮箱的名称。虽然消息是从RabbitMQ到你的应用之间进行传递，但是他们只能被保存到一个队列中。一个队列只能被主机的内存和硬盘大小所限制，它本质上是一个大的消息缓冲区。很多生产者都可以发送消息到一个队列中，并且很多消费者可以尝试从一个队列中接收数据。下面是我们如何表示一个队列。


  [1]: https://raw.githubusercontent.com/hongbochen/mks/master/images/producer.png