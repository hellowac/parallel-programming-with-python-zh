# 根据任务类型定义队列

负责计算 `Fibonacci` 的任务已实现并正在运行。 我们可以看到所有任务都被发送到 `Celery` 的默认队列中。 但是，有多种方法可以将任务路由到不同的队列； 让我们在服务器端重构我们的架构，并从客户端实现所谓的路由任务。 我们将为每种类型的任务指定队列。

在服务器端启动`Celery`服务器的那一刻，我们会建立三个不同的队列。 这些现在将被worker看到和消费。 `Fibonacci` 任务的队列是 `fibo_queue`，平方根任务的 `sqrt_queue`，Web 爬虫任务的 `webcrawler_queue`。 但是，将任务分开有什么好处呢？ 让我们列举一些：

- 它将相同类型的任务分组，使它们的监控更容易
- 它定义了专用于消费特定队列的`worker`，从而提高了性能
- 它在具有性能更好的机器上建立任务更重的队列

!!! info ""

    前面几点本书不会解释，但我们可以通过初始化 Celery 服务器甚至在网络中分配具有专用队列的代理来实现负载平衡。 我建议您使用 Celery 尝试这种集群风格。

要在服务器中设置队列，我们只需要使用以下命令启动 `Celery`：

```shell
$celery –A tasks –Q sqrt_queue,fibo_queue,webcrawler_queue worker --loglevel=info
```

下图是在服务端截图：

```log
-------------- [queues]
                .> fibo_queue       exchange=fibo_queue(direct) key=fibo_queue
                .> sqrt_queue       exchange=sqrt_queue(direct) key=sqrt_queue
                .> webcrawler_queue exchange=webcrawler_queue(direct) key=webcrawler_queue
```

在转到下一个示例之前，让我们将现有任务的发送路由到它们的队列。 在服务器端，在 `task_dispatcher.py` 模块中，我们将更改 `send_task` 调用，以便下次分派任务时，它们将被定向到不同的队列。 我们现在将按如下方式更改 `sqrt_task` 调用：

```python
app.send_task('tasks.sqrt_task', args=(value,), queue='sqrt_queue', routing_key='sqrt_queue')
```

然后，修改`fibo_task`调用，如下：

```python
app.send_task('tasks.fibo_task', args=(x,), queue='fibo_queue', routing_key='fibo_queue')
```

!!! info ""

    如果有兴趣监控队列、统计任务数量或者其他，请参考`Celery`文档 <http://celery.readthedocs.org/en/latest/userguide/monitoring.html>{target="_blank"}。

    在任何情况用`Redis`，`redis-cli`都可以作为一个工具。队列、任务、workders都可以被监控，详见 <http://celery.readthedocs.org/en/latest/userguide/monitoring.html#workers>{target="_blank"}.
