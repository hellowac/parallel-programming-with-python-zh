# 理解Celery架构

Celery架构基于可插拔组件和用*message transport(broker)*实现的消息交换机制。具体的如下图所示：

![1](../imgs/7-01.png)

现在，让我们详细的介绍Celery的每个组件。

## 处理任务

在上图中的*client*组件，有创建和分派任务到brokers的方法。

分析如下示例代码来演示通过使用`@app.task`装饰器来定义一个任务，它可以被一个**Celery**应用的实例访问，下面代码展示了一个简单的`Hello World app`：

```python
 @app.task
 def hello_world():
  return "Hello I'm a celery task"
```

!!! info ""

    任何可以被调用的方法或对象都可以成为任务 (Any callable can be a task.)

正如我们之前所说，有各种类型的任务：同步、异步、周期和计划任务。当我们调用任务，它返回类型AsyncResult的实例。AsyncResult对象中可以查看任务状态，当任务结束后可以查看返回结果。然而，为了利用这个机制，另一个叫做*result backend*的组件必须启动，这将在本章的后面讲解。为了分派任务，我们可以用下面这些方法：

- `delay(arg, kwarg=value)` : 它会调用`apply_aync`方法。
- `apply_async((arg,), {'kwarg': value})` : 该方法可以为任务设置很多参数，一些参数如下。
   - `countdown` : 默认任务是立即执行，该参数设置经过`countdown`秒之后执行。
   - `expires` : 代表经过多长时间终止。
   - `retry` : 如果连接或者发送任务失败，该参数可是重试。
   - `queue` : 任务队列。
   - `serializer` : 数据格式，其他还有json、yaml等等
   - `link` : 连接一个或多个即将执行的任务。
   - `link_error` : 连接一个或多个执行失败的任务。
   - `apply((arg,), {'kwarg': value})` : 在本地进程以同步的方式执行任务，因而阻塞直到结果就绪。

!!! info ""

    Celery 提供了查看任务状态的机制，这在跟踪进程的真实状态非常有用。更多的关于内建任务状态的资料请查看<http://celery.readthedocs.org/en/latest/reference/celery.states.html>

## 消息转发(broker)

`broker`是**Celery**中的核心组件，通过`broker`可以发送和接受消息，来完成和`workers`的通信。**Celery**支持大量的`brokers`。然而，对于某些`broker`，不是所有的**Celery**机制都实现了。实现功能最全的是**RabbitMQ**和**Redis**。在本书中，我们将采用**Redis**作为`broker`。`broker`提供在不同客户端应用之间通信的方法，客户端应用发送任务，`workers`执行任务。可以有多台带有`broker`的机器等待接收消息，然后发送消息给`workers`。

## 理解workers

`Workers`负责执行接收到的任务。**Celery**提供了一系列的机制，我们可以选择最合适的方式来控制`workers`的行为。这些机制如下：

- **并发模式**(Concurrency mode)：例如进程、线程、协程(Eventlet)和Gevent
- **远程控制**(Remote control)：利用该机制，可以通过高优先级队列发送消息到某个特定的worker来改变行为，包括runtime。
- **撤销任务**(Revoking tasks)：利用该机制，可以指挥一个或多个workers来忽略一个或多个任务。

更多的特性可以在运行时设定或者改变。比如，**worker**在某一段时间执行的任务数，**worker**在哪个队列消费等等。关于**worker**更多的信息可以参考  <http://docs.celeryproject.org/en/latest/userguide/workers.html#remote-control>{target="_blank"}

## 理解result backends

**Result backend**组件存储任务的状态和任务返回给客户端应用的结果。**Celery**支持的**result backend**之中，比较出彩的有 **RabbitMQ**, **Redis**, **MongoDB**, **Memcached**。每个**result backend**都有各自的优缺点，详见  <http://docs.celeryproject.org/en/latest/userguide/tasks.html#task-result-backends>{target="_blank"}

现在，我们对**Celery**架构有了一个大概的认识。下面我们建立一个开发环境来实现一些例子。
