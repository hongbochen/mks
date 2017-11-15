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


![Queue][2]

消费者(`Consumer`)与接收有相似的意义。一个消费者是等待来接受消息的程序。

![Consumer][3]

注意，生产者，消费者和代理不必放到同一个主机上；在大多数应用中确实没有这样。

### *“Hello World”*
**(使用Java客户端)**

在这一部分，我们将会使用Java写两个程序：一个发送单一消息的生产者，和一个接收消息并打印他们的消费者。我们将会在Java API中解释一些细节，关注于这些简单的事情来开始。这就是消息的"Hello World"


  [1]: https://raw.githubusercontent.com/hongbochen/mks/master/images/producer.png
  [2]: https://raw.githubusercontent.com/hongbochen/mks/master/images/queue.png
  [3]: https://raw.githubusercontent.com/hongbochen/mks/master/images/consumer.png