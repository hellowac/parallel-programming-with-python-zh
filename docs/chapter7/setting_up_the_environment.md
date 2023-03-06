# 建立环境

在本节中，我们将在 `Linux` 中设置两台机器。 第一个，主机名 `foshan`，将执行客户端角色，应用程序 `Celery` 将在其中调度要执行的任务。 另一台主机名为 `Phoenix` 的机器将执行**代理**（broker）、**结果后端**（result backend）和worker使用的队列的角色。

## 配置客户端机器

让我们开始设置客户端机器。 在这台机器上，我们将使用 `pyvenv` 工具设置一个 `Python 3.3` 的虚拟环境。 `pyvenv` 的目标是不使用额外的模块污染操作系统中存在的 `Python`，而是将每个项目所需的开发环境分开。 我们将执行以下命令来创建我们的虚拟环境：

 ```shell
 $pyvenv celery_env
 ```

上述命令在当前路径创建一个名为`celery_env`的文件夹，里面包含所有Python开发环境必须的结构。下图是该目录所包含的内容：

```shell
# 这里使用的最新的python venv模块
$# ./Python-3.9.14/python -m venv celery_env
$# ls celery_env/
bin  include  lib  lib64  pyvenv.cfg
```

在创建了虚拟环境之后，我们就可以开始工作并安装需要使用的包。然而，首先我们得激活这个环境，执行以下命名：

```shell
$# source celery_env/bin/activate
```

当命令行提示符改变了，例如在左边出现`celery_env`，就说明激活完成。所有你安装的包都只在这个目录下有效，而不是在整个系统中有效。

```shell
(celery_env) $# ls celery_env/
bin  include  lib  lib64  pyvenv.cfg
```

!!! info ""

    用`--system-site-packages`标识可以创建能够访问系统`site-packages`的虚拟环境，但是不推荐使用。

现在，我们有一个虚拟环境，假设已经安装好了`setuptools`或者`pip`。下面为客户端安装必须的包，如下命令：

```shell
$pip install celery
```

下图是已经安装好的framework v3.1.9，将在本书中使用该版本。

```shell
# 由于当前(2023)python2已不再支持，顾这里安装的最新版本v5.2.7
(celery_env) $# python
Python 3.9.14 (main, Sep 19 2022, 12:04:09)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-44)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import celery
>>> celery.VERSION
version_info_t(major=5, minor=2, micro=7, releaselevel='', serial='')
>>>
```

现在我们要在**Celery**中安装支持的**Redis**，这样客户端就可以通过`broker`传输消息了。用如下命令：

```shell
$pip install celery[redis]
```

现在我们的客户端环境配置好了，在开始编码之前，我们必须配置好服务器端的环境。

## 配置服务器

为了配置服务器，我们首先安装**Redis**，**Redis**将作为`broker`和`result backend`。使用如下命令：

```shell
$sudo apt-get install redis-server
```

启动Redis：

```shell
$redis-server
```

如果成功，会出现类似下图中的输出

```log
2905:C 06 Mar 15:53:46.571 * supervised by systemd, will signal readiness
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.2.12 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 2905
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

2905:M 06 Mar 15:53:46.574 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
2905:M 06 Mar 15:53:46.574 # Server started, Redis version 3.2.12
2905:M 06 Mar 15:53:46.574 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
2905:M 06 Mar 15:53:46.574 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
2905:M 06 Mar 15:53:46.574 * The server is now ready to accept connections on port 6379
```
