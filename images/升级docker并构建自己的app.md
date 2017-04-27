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

安装完成之后，我们通过运行`hello-world`镜像来验证Docker CE是否安装完成。

### 卸载Docker

我们可以使用下面的命令来卸载Docker包。

```

sudo apt-get purge docker-ce

```

但是镜像，容器，卷或者是定制的配置文件将不会自动被删除，我们可以通过下面的命令来移走：

```

sudo rm -rf /var/lib/docker

```

## （二）、构建新的app

### 你的新的开发环境

在过去，如果你想要开始编写一个Python应用，第一步工作便是在你的机器上安装一个Python运行时环境，但是这样便会导致一个问题是，如果想要你的应用如期运行你的机器环境在哪里，还有运行你的应用的app在哪里。

使用Docker，你仅仅需要抓取一个轻便的Python运行时作为一个镜像，不必安装。然后，你的构建需要包括基本的Python镜像和你的app代码，确保你的app，他的依赖和运行时都在一块。

这些轻便的镜像被定义称为`Dockerfile`。

### 使用`Dockerfile`定义一个容器

`Dockerfile`将会定义在你的容器环境中什么将会运行。访问的资源例如网络接口和磁盘驱动在环境中都被虚拟出来了，这个是和你系统中的其他事物隔离的，所以你必须映射端口到外面的世界，并且必须要指定那些文件你想要复制到那个环境中。然后，完成那些操作之后，你可能希望定义在这个`Dockerfile`中的app的构建在任何地方运行都一样的。

