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

#### `Dockerfile`

创建一个空的目录然后把这个名称为`Dockerfile`的文件放进去。注意每一行的注释。

```

# 使用一个官方的Python运行时作为一个镜像
FROM python:2.7-slim

# 设置工作目录到/app
WORKDIR /app

# 复制当前目录内容到/app的容器中
ADD . /app

# 安装指定在requirements.txt文件中的所有的包
RUN pip install -r requirements.txt

# 使80端口在容器外能够被访问
EXPOSE 80

# 定义环境变量
ENV NAME World

# 当容器启动的时候运行app.py
CMD ["python", "app.py"]

```

这个`Dockerfile`引用了一些我们没有创建的文件，`app.py`和`requirements.txt`。在下面我们获取这些文件。

### app本身

获取这两个文件并放到和`Dockerfile`相同的目录中。这样就完成了我们的app，看上去是很简单的。当上述的`Dockerfile`被构建进入镜像的时候，`app.py`和`requirements.txt`也将会存在，因为`Dockerfile`的`ADD`命令，`app.py`的输出将通过HTTP被访问，因为`EXPOSE`命令。

#### `requirements.txt`


```

Flask
Redis

```

#### `app.py`

```

from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr('counter')
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv('NAME', "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
	app.run(host='0.0.0.0', port=80)

```


现在我们可以看到`pip install requirements.txt`安装了Flask和Redis库，并且app打印出了环境变量`NAME`，和`socker.gethoustname()`调用后的输出是一样的。最后，因为Redis并没有运行（因为我们仅仅安装了Python库，而不是Redis本身），所以我们得到的结果将会是尝试使用它但是失败了，并且产生一个错误信息。

### 构建app

就是这样，在你的系统中，你不需要Python或者是`requirements.txt`文件中的任何东西，你也不需要构建或者是运行这个景象安装到你的系统中。看上去好像是你设置了Python或者是Flask的环境变量，实际上并没有。下面是`ls`应该显示的：

```

$ ls
Dockerfile		app.py			requirements.txt

```

现在运行build命令。这将会创建一个Docker镜像，该镜像我们使用`-t`选项来标记他，这样他就有了一个友好的名称。

```

docker build -t friendlyhello .

```

你构建的镜像在哪呢?他在你的机器的本地Docker镜像条目中。

```

$ docker images

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398

```

### 运行app

运行app，使用`-p`选项映射你的机器的端口4000到容器的`EXPOSE`的端口80。

```

docker run -p 4000:80 friendlyhello

```

你应该看到一个提醒，Python正在`http://0.0.0.0:80`服务你的app。但是那个信息是来自内部容器的，他并不知道你映射了端口80到4000了，使得当前的URL `http://localhost:4000` 。到这里，你将会看到`Hello World`文本，容器ID和Redis错误信息。

	注意：这个端口映射`4000:80`是为了证明在`Dockerfile`中你的`EXPOSE`和你使用`docker run -p`发布的不同的。在后面的步骤中，我们仅仅映射端口80到端口80，这样就可以使用`http://localhost`。
	
在你的终端中使用`CTRL+C`来退出。

现在我们在后台运行该app，使用分离的模式：

```

docker run -d -p 4000:80 friendlyhello

```

你获取到了一个长得容器ID并且然后被踢回到你的终端中。你的容器正在后台运行。你可以使用`docker ps`来看到一个简便的容器ID.

```

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago

```

你将看到`CONTAINER ID`和`http://localhost:4000`中的是相匹配的。

现在使用`docker stop`来结束进程，使用`CONTAINER ID`，就像下面这样：

```

docker stop 1fa4ab2cf395

```

### 共享你的镜像

就像我们在上一篇中讲解的一样，使用下面的命令来共享您的镜像：

```

docker login
docker tag friendlyhello username/repository:tag
docker push username/repository:tag
docker run -p 4000:80 username/repository:tag

```