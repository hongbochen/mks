---
title: 在RabbitMQ上的访问控制(身份认证，授权)
tags: RabbitMQ, 访问控制, 身份认证, 授权
grammar_cjkRuby: true
---

在安装RabbitMQ Server那一篇文章中，我们介绍到，RabbitMQ Server有一个默认的`guest`用户，但是只能在`localhost`访问的时候才能有效。所以，由于我们是把RabbitMQ放到不同的服务器上，所以必须要配置其访问控制来达到访问的效果。

在这一篇文章中，我们将会介绍实现访问控制的身份认证和授权。