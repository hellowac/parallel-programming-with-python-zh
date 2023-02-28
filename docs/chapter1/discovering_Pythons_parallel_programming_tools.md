# 发现Python并行编程的工具

由 Guido Van Rossum 创建的 Python 语言是一种多范式、多用途的语言。 由于其强大的简单性和易于维护，它已在世界范围内被广泛接受。 它也被称为包含电池的语言。 模块种类繁多，使用起来更顺畅。 在并行编程中，Python 具有简化实现的内置和外部模块。 本书是基于Python3.X的。

## Python的threading模块

Python的**threading**模块为模块 **_thread** 提供了一个抽象层，它是一个较低级别的模块。 它提供的功能可以帮助程序员完成基于线程开发并行系统的艰巨任务。 **threading**模块的官方文档可以在<http://docs.python.org/3/library/threading.html?highlight=threading#module-threadin>{target="_blank"}中找到。

## Python的mutliprocess模块

**multiprocessing** 模块旨在为使用基于进程的并行性提供一个简单的 API。 该模块类似于线程模块，它简化了进程之间的交替，没有太大的困难。基于进程的方法在 Python 用户社区中非常流行，因为它是回答有关使用 CPU 绑定线程和 Python 中存在的 GIL 的问题的替代方法。 **multiprocessing**模块的官方文档可以在以下位置找到:<http://docs.python.org/3/library/multiprocessing.html?highlight=multiprocessing#multiprocessing>

## Python的parallel模块

**parallel Python** 是外部模块，它提供了丰富的 API，这些API利用进程的方法创建并行和分布式系统。该模块是轻量级并且易安装的，并可与其他 Python 程序集成。 可以在 <http://parallelpython.com> 找到 **parallel Python** 模块。 在所有功能中，我们可能会强调以下内容：

* 自动检测最佳配置
* 在运行时可以更改许多工作进程的状态
* 动态的负载均衡
* 容错性
* 自动发现计算资源

## Celery分布式任务队列

**Celery** 是一个出色的 Python 模块，用于创建分布式系统并具有出色的文档。 它在并发形式上使用了至少三种不同类型的方法来执行任务——multiprocessing、Eventlet 和 Gevent。 然而，这项工作将集中精力于多处理方法的使用。 而且，只需要通过配置就能实现进程间的互相通信，它将作为一项课题研究，以便读者能够与他/她自己的实验进行比较。

Celery模块可以在官方的项目页面<http://celeryproject.org>{target="_blank"}得到。
