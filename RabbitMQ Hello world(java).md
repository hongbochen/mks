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