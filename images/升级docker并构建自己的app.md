---
title: 升级docker并构建自己的app
tags: docker,app
grammar_cjkRuby: true
---

现在docker已经升级了很多版本了，而我目前的docker版本仍然是`1.12.5`，比较老的版本，所以现在我们需要升级我们的docker版本，首先是如何查看我们系统中的docker版本呢？

运行命令`docker --version`即可查看。

## （一）、Docker新版本介绍及安装

目前，Docker分为了两个可用的版本，分别为`Docker企业版`和`Docker社区版`，故名思议，`Docker EE`，即Docker企业版是专门为企业开发和IT团队构建，部署商业应用所设计的，而`Docker CE`，即Docker社区本是为开发者和刚开始使用Docker的小团队开发设计的。在这里我们使用`Docker CE`。


### 卸载老版本

Docker的老版本被称为`docker`或者是`docker engine`，如果安装过，先把他们卸载掉。

```

 sudo apt-get remove docker docker-engine

```

如果卸载成功，`apt-get`将会报出没有安装包被安装。

位于目录`/var/lib/docker/`中的镜像，容器，卷和网络被保留。Docker CE包现在被称为`docker-ce`，Docker EE包现在被称为`docker-ee`。

### 为Trusty 14.04推荐的额外的包

如果没有其他原因，推荐您安装`linux-image-extra-*`这些包，这些允许Docker使用`aufs`存储驱动。

```

$ sudo apt-get update

$ sudo apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual

```

### 安装Docker

有两种安装方式，一个是通过仓库安装，一个是通过包安装，在这里我们选择使用包安装。

我们需要下载`.deb`文件来安装docker。这里有一个麻烦的地方就是，每次我们想要更新docker的话，就需要下载一个新的文件，无所谓了，我们选择包安装。

`Docker CE`和`Docker EE`的安装步骤也是不一样的，我们只关注`Docker CE`。进入[下载地址][1]


  [1]: https://download.docker.com/linux/ubuntu/dists/
  
  ，选择我们的ubuntu版本，浏览`pool/stable`，选择`amd64`或者是`armhf`，然后下载相应你想要安装的的`.deb`文件。
  
  下载完成之后，使用下面的命令来安装你所下载的安装包。
  
```

dpkg -i /path/to/package.deb

```