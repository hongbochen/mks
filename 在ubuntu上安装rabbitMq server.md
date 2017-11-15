---
title: 在Ubuntu上安装rabbitMq server
tags: rabbitmq server,rabbitmq,ubuntu
grammar_cjkRuby: true
---

### 下载Server

| 描述        | 下载           | 
| ------------- |:-------------:| 
| 基于debian的linux的安装包(github)      | [rabbitmq-server_3.6.13-1_all.deb][1] |
| 基于debian的linux的安装包(Bintray)       | [rabbitmq-server_3.6.13-1_all.deb][2]      |

### 标准Ubuntu和Debian仓库

`rabbitmq-server`被包含在标准Debian和Ubuntu仓库中。然而，包含的版本经常是过旧的。你可以从`rabbitmq.com`或者是`Package Cloud`中的apt仓库中来安装最新的安装包。检查[Debian包管理][3]和Ubuntu包管理来获取哪一个发行版的版本是可用的。

你也可以下载上面链接中的安装包并且使用`dpkg`安装，或使用官方的APT仓库（见下面）。

### 支持的发布

下面是RabbitMQ 3.6.3 支持的基于Debian发行版的列表。

 - Ubuntu 14.04到17.02
 - Debian Jessie
 - Debian Wheezy

如果是依赖满足的话，该安装包可能也可以正常工作在其他基于Debian的发行版上，但是他们的测试和支持是尽力而为的。

### 安装Erlang/OTP

RabitMQ需要Erlang/OTP来运行。在标准Debial和Ubuntu中的Erlang/OTP也是过时的。考虑安装一个新的版本，例如19.3。


| Erlang发行序列        | 提供他的仓库           | 
| ------------- |:-------------:| 
| 20.x     |[Erlang解决方案][4] 。从3.6.11开始支持，以前版本不支持 |
| 19.x     |[Erlang解决方案][5]。Ubuntu Zesty(17.04) |
| 18.x     |[Erlang解决方案][6]。Ubuntu Yakkety(16.10),Ubuntu Xenial(16.04) |
| 17.x     |[Erlang解决方案][7]。Debian Jessie, Debian Wheezy backports |

#### Erlang版本固定

`apt包固定`可以被使用来避免不期望的Erlang更新。下面引用的文件的例子将会固定`esl-erlang`包到`19.3.6`并且`erlang-*`包到`19.3`。

```

	# /etc/apt/preferences.d/erlang
	Package: erlang*
	Pin: version 1:19.3-1
	Pin-Priority: 1000

	Package: esl-erlang
	Pin: version 1:19.3.6
	Pin-Priority: 1000

```

上面的例子应该被放到`/etc/apt/preference.d/`下面的一个文件中，例如`/etc/apt/preference.d/erlang`。

使用下面的命令可以使有效的包锁定被验证。

```
	sudo apt-cache policy

```

### 包依赖

当使用apt安装的时候，所有的依赖在最近的发行中都应该自动满足。在不满足的情况下，所有的依赖包应该从一个合适的仓库中可用。然而，当使用dpkg安装的时候，就不会出现这样的情况。下面是RabbitMQ服务例如3.6.3的依赖列表。

```

	erlang-nox (>= 1:16.b.3) | esl-erlang. Erlang can installed either from the standard repositories, backport repositories or Erlang Solutions.
	init-system-helpers >= 1.13. Required for systemd support.
	socat
	adduser
	logrotate

```

### APT仓库

RabbitMQ维护自己的APT仓库。我们在`Package Cloud`中也有可选择的仓库。

#### 使用rabbitmq.com APT仓库

执行下面的命令将APT仓库添加到你的`/etc/apt/sources.list.d`中。

```

	echo 'deb http://www.rabbitmq.com/debian/ testing main' |
     sudo tee /etc/apt/sources.list.d/rabbitmq.list

```

(请注意单词在这行中的`testing`指向我们RabbitMQ的发行版的状态，不是任何特定的Debian发行版。你可以使用Debian稳定的，测试的或不稳定的或者是Ubuntu来用他。我们使用"testing"来描述发行版来强调我们发行的有些频繁。)

（可选的）为了避免未签名包的警告，使用`apt-key(8)`来添加我们的[公共key][8]到你的可信的key列表中。

```

	wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc |
     sudo apt-key add -

```

我们公共的签名key从[Bintray][9]中也是可用的。

```

	wget -O- https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc |
		 sudo apt-key add -

```

运行下面的命令来更新包列表。

```

	sudo apt-get update

```

安装`rabbitmq-server`包。

```

sudo apt-get install rabbitmq-server

```

### 运行RabbitMQ服务器

#### 定置化RabbitMQ环境变量

服务器应该使用默认值启动的。你可以[定制化RabbitMQ环境][10]。也可以去查看如何[配置组件][11]。

#### 开启服务

当RabbitMQ服务器包被安装的时候，该服务默认是以守护进程启动的。

作为一个管理员，在Debian中使用`service rabbitmq-server start`启动和停止服务。

*注意：该服务被设置为以系统用户`rabbitmq`来运行。如果你改变了节点数据库或日志的位置，你必须确定文件是属于这个用户的（并且更新环境变量）。*

### 端口访问

SELinux，或者是类似的机制可能会阻止RabbitMQ绑定端口。当这种事情发生的时候，RabbitMQ将会启动失败。防火墙会阻止节点和CLI工具之间的通信。确保下面的端口是开着的:

 - 4369：epmd，一个被RabbitMQ节点和CLI工具使用的伙伴搜索服务。
 - 5672，5671：被带有或不带有TLS的AMQP 0-9-1和1.0客户端使用的端口。
 - 25672：被Erlang发行版用于内部节点和CLI工具通信并且从一个动态的范围被分配（默认限制到一个单一的端口，以AMQP端口+20000来计算）。
 - 15672：HTTP API客户端和rabbitmqadmin(仅仅管理插件被使能的时候)
 - 61613，61614：带有或不带有TLS的STOMP客户端（仅仅在STOMP插件被使能的时候）
 - 1883,8883：（如果MQTT插件被使能的时候，带有或不带有TLS的MQTT客户端。）
 - 15674：STOMP-over-WebSocket客户端（仅仅Web STOMP插件被使能的时候）
 - 15675：MQTT-over-WebSocket客户端（仅仅是Web MQTT 插件被使能的时候）

### 默认用户访问

该代理使用密码`guest`创建了一个用户`guest`。未配置的客户端大体上将会使用这些证书。默认情况下，这些证书只有作为localhost连接的时候才会被使用，所在连接其他机器之前你需要做一些操作。

查看文档[访问控制][12]来获取更多信息来创建更多用户，删除`guest`用户，或者是对`guest`用户允许远程访问。

### 在Linux中控制系统限制

运行生产工作负载的RabbitMQ的安装可能需要系统限制和内核参数整定，来处理一定数量的并行连接和队列。需要满足的主要设置就是打开文件的最大数量，也被称为`ulimit -n`。在很多操作系统中的默认值对于消息代理来说太低了（例如在多个Linux发行版上是1024）。我们推荐在生产环境中对用户`rabbitmq`最少允许65536个文件描述符。4096对大多数开发工作负载来说应该是充足的。

#### 使用systemd（最近Linux发行版）

在发行版中使用systemd。系统限制是由位于`/etc/systemd/system/rabbitmq-server.service.d/limits.conf`文件来控制的。例如：

```

	[Service]
	LimitNOFILE=300000

```

#### 没有systemd(较早期的Linux发行版)

在发行版中，使RabbitMQ能够调整每一个用户限制最直接的方式就是不使用systemd而是在服务启动之前，编辑`/etc/default/rabbitmq-server `(RabbitMQ Debian包提供的)或者是`rabbitmq-env.conf`来唤起ulimit。

```

	ulimit -S -n 4096

```

该软件limit不能超过硬限制（在大多数发行版中默认为4096）。硬限制可以通过文件`/etc/security/limits.conf`来增加。这也要求使能`pam_limits.so`模块，并且重启重新登录系统。

注意在运行的OS进程中,limits是不能被改变的。

#### 验证limit

RabbitMQ管理页面展示了可用的文件描述符的数量。

```

	rabbitmqctl status

```
包含同样的值。

下列命令：

```

	cat /proc/$RABBITMQ_BEAM_PROCESS_PID/limits

```

可以被用于显示一个运行的进程的有限的限制(limits)。`$RABBITMQ_BEAM_PROCESS_PID`是运行RabbitMQ的Erlang VM的操作系统PID,就像是`rabbitmqctl status`返回的值。

#### 配置管理工具

配置管理工具提供了系统限制调整的协助。我们的开发工具指导列出了相关的模块和项目。

### 管理代理

停止服务或者是检查他的状态等，你可以使用包特定脚本或者是运行rabbitmqctl命令。在系统路径中他是可用的。所有的rabbitmqctl命令将会报告节点缺少，如果没有代理正在运行的话。

运行`rabbitmqctl stop`来停止服务。

运行`rabbitmqctl status`来检查其是否在运行。

#### 日志

来自服务的所有的输出被送到位于RABBITMQ_LOG_BASE目录中的RABBITMQ_NODENAME.log文件中。其他的日志数据被写到RABBITMQ_NODENAME-sasl.log中。

代理经常依赖于日志文件，所有一个完整的日志历史被维护。

你可以使用轮替程序来做所有必要的循环和压缩，并且你可以修改它。默认情况下，这个脚本没有运行默认位于`/var/log/rabbitmq`目录中的文件。查看`/etc/logrotate.d/rabbitmq-server`来配置轮替。


  [1]: https://github.com/rabbitmq/rabbitmq-server/releases/download/rabbitmq_v3_6_13/rabbitmq-server_3.6.13-1_all.deb
  [2]: https://dl.bintray.com/rabbitmq/rabbitmq-server-deb/rabbitmq-server_3.6.13-1_all.deb
  [3]: http://packages.qa.debian.org/r/rabbitmq-server.html
  [4]: https://packages.erlang-solutions.com/erlang/#tabs-debian
  [5]: https://packages.erlang-solutions.com/erlang/#tabs-debian
  [6]: https://packages.erlang-solutions.com/erlang/#tabs-debian
  [7]: https://packages.erlang-solutions.com/erlang/#tabs-debian
  [8]: http://www.rabbitmq.com/rabbitmq-release-signing-key.asc
  [9]: https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
  [10]: http://www.rabbitmq.com/configure.html#customise-general-unix-environment
  [11]: http://www.rabbitmq.com/configure.html#configuration-file
  [12]: http://www.rabbitmq.com/access-control.html