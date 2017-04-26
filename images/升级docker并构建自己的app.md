---
title: 升级docker并构建自己的app
tags: docker,app
grammar_cjkRuby: true
---

现在docker已经升级了很多版本了，而我目前的docker版本仍然是`1.12.5`，比较老的版本，所以现在我们需要升级我们的docker版本，首先是如何查看我们系统中的docker版本呢？

运行命令`docker --version`即可查看。

（一）、Docker新版本介绍及安装

目前，Docker分为了两个可用的版本，分别为`Docker企业版`和`Docker社区版`，故名思议，`Docker EE`，即Docker企业版是专门为企业开发和IT团队构建，部署商业应用所设计的，而`Docker CE`，即Docker社区本是为开发者和刚开始使用Docker的小团队开发设计的。在这里我们使用`Docker CE`。
