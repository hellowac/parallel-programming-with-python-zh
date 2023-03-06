# 使用 Celery 获取斐波那契数列项

让我们再次去分配我们的多个输入，以计算第 `n` 个斐波那契项，每个项都以分布式方式计算。 计算 `Fibonacci` 的函数相对于前面的章节会有一些变化。 变化很小； 现在我们有了 `@app.task` 装饰器和返回消息中的一个小改动。

在代理所在的服务器计算机中的 `tasks.py` 模块（之前创建）中，我们将停止执行 `Celery（Ctrl + C`）并添加 `fibo_task` 任务。 这是通过使用以下代码完成的：

```python
@app.task
def fibo_task(value):
    a, b = 0,1
    for item in range(value):
        a, b = b, a + b
    message = "The Fibonacci calculated with task id %s was %d" % (fibo_task.request.id, a)
    return (value, message)
```

通过`ask.reaquest.id`得到任务的ID，请求对象是`task`的对象，`task`对象提供了`task`执行的上下文。通过上下文可以得到`task`的ID等信息。

在`tasks.py`模块加入了新的任务之后，再一次初始化**Celery**，结果如下图：

```log
- *** --- * --- .> concurrency: 2 (prefork)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** -----
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery


[tasks]
  . tasks.fibo_task
  . tasks.sqrt_task

[2023-03-06 17:05:34,402: INFO/MainProcess] Connected to redis://localhost:6379/0
```

现在我们把`fibo_task`任务装载到**Celery server**，我们将在客户端实现对该任务的调用。

在`task_dispatcher.py`模块，我们会申明`input_list`，如下：

```python
input_list = [4, 3, 8, 6, 10]
```

和前面的做法一样，定义`manage_fibo_task`方法：

正如我们在上一节创建的 `sqrt_task` 任务中所做的那样，我们将创建一个方法来组织我们的调用而不污染 `__main__` 块。 我们将此函数命名为 `manage_fibo_task`。 如以下实现：

```python
def manage_fibo_task(value_list):
    async_result_dict = {x: app.send_task('tasks.fibo_task',
        args=(x,)) for x in value_list}

    for key, value in async_result_dict.items():
        logger.info("Value [%d] -> %s" % (key, value.get()[1]))
```

在 `manage_fibo_task` 函数中，我们创建了一个名为 `async_result_dict` 的字典，填充相同的键值对。 `key` 是作为参数传递的项，用于获取 `Fibonacci` 的无数项，`value` 是从调用 `send_task` 方法返回的 `AsyncResult` 的实例。 通过这种方法，我们可以监控任务的状态和结果。

最后，遍历字典得到输入值和输出结果并封装成字典。`AsyncResult`类的`get()`函数可以让我们获取处理结果。

`get()`方法会阻塞进程。一个好的方法是调用`ready()`方法来检查结果是否返回了。

可能会注意到 `get()` 函数可能不会立即返回结果，因为处理仍在进行。 在客户端调用 `get()` 方法可以阻止调用之后的处理。 将调用结合到 `ready()` 方法是个好主意，这样可以检查是否准备好获取结果。

因此，结果展示循环可以修改为如下代码:

```python
for key, value in async_result_dict.items():
    if value.ready():
        logger.info("Value [%d] -> %s" % (key, value.get()[1]))
    else:
        logger.info("Task [%s] is not ready" % value.task_id)
```

Depending on the type of task to be executed, there may be a considerable delay in the result. Therefore, by calling get() without considering the return status, we can block the code running at the point where the get() function was called. To tackle this, we should define an argument called timeout in the get(timeout=x) method. So, by minimizing this blocking, we can prevent tasks from having problems in returning results, which would impact the running of the execution for an indefinite time.

根据要执行的任务类型，结果可能会有相当长的延迟。 因此，通过调用 `get()` 而不考虑返回状态，我们可以阻止代码运行在 `get()` 函数被调用的地方。 为了解决这个问题，我们应该在 `get(timeout=x)` 方法中定义一个名为 `timeout` 的参数。 因此，通过最小化这种阻塞，我们可以防止任务在返回结果时出现问题，这会无限期地影响执行的运行。

最后，我们添加了对 `manage_fibo_task` 函数的调用，作为参数传递给我们的 `input_list`。 代码如下：

```python
if __name__ == '__main__':
    #manage_sqrt_task(4)
    manage_fibo_task(input_list)
```

当我们执行`task_dispatcher.py`中的代码时，可以在旁边看到如下输出服务器：

```shell
$# python task_dispatcher.py
2023-03-06 17:20:38,902 - Value [4] -> The Fibonacci calculated with task id 03328f4d-8226-4a15-853d-8b8ab5833b72 was 3
2023-03-06 17:20:38,904 - Value [3] -> The Fibonacci calculated with task id 4ea527de-0a96-4c6c-ac25-8e4a01a0e919 was 2
2023-03-06 17:20:38,906 - Task [448d8127-763c-4025-84a1-e05e9979841a] is not ready
2023-03-06 17:20:38,909 - Task [f639a24f-cbf5-403d-b243-4c5c54f5b77a] is not ready
2023-03-06 17:20:38,909 - Task [4e2999a7-bae8-454c-9f70-8b513dd0844e] is not ready
```

在客户端有如下输出：

```shell
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** -----
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery


[tasks]
  . tasks.fibo_task
  . tasks.sqrt_task

[2023-03-06 17:20:35,572: INFO/MainProcess] Connected to redis://localhost:6379/0
[2023-03-06 17:20:35,578: INFO/MainProcess] mingle: searching for neighbors
[2023-03-06 17:20:36,599: INFO/MainProcess] mingle: all alone
[2023-03-06 17:20:36,616: INFO/MainProcess] celery@ch1.nauu.com ready.
[2023-03-06 17:20:38,859: INFO/MainProcess] Task tasks.fibo_task[03328f4d-8226-4a15-853d-8b8ab5833b72] received
[2023-03-06 17:20:38,877: INFO/MainProcess] Task tasks.fibo_task[4ea527de-0a96-4c6c-ac25-8e4a01a0e919] received
[2023-03-06 17:20:38,884: INFO/MainProcess] Task tasks.fibo_task[448d8127-763c-4025-84a1-e05e9979841a] received
[2023-03-06 17:20:38,890: INFO/MainProcess] Task tasks.fibo_task[f639a24f-cbf5-403d-b243-4c5c54f5b77a] received
[2023-03-06 17:20:38,896: INFO/MainProcess] Task tasks.fibo_task[4e2999a7-bae8-454c-9f70-8b513dd0844e] received
[2023-03-06 17:20:38,898: INFO/ForkPoolWorker-2] Task tasks.fibo_task[03328f4d-8226-4a15-853d-8b8ab5833b72] succeeded in 0.03570139221847057s: (4, 'The Fibonacci calculated with task id 03328f4d-8226-4a15-853d-8b8ab5833b72 was 3')
[2023-03-06 17:20:38,903: INFO/ForkPoolWorker-1] Task tasks.fibo_task[4ea527de-0a96-4c6c-ac25-8e4a01a0e919] succeeded in 0.02462736703455448s: (3, 'The Fibonacci calculated with task id 4ea527de-0a96-4c6c-ac25-8e4a01a0e919 was 2')
[2023-03-06 17:20:38,920: INFO/ForkPoolWorker-2] Task tasks.fibo_task[448d8127-763c-4025-84a1-e05e9979841a] succeeded in 0.014719393104314804s: (8, 'The Fibonacci calculated with task id 448d8127-763c-4025-84a1-e05e9979841a was 21')
[2023-03-06 17:20:38,928: INFO/ForkPoolWorker-2] Task tasks.fibo_task[4e2999a7-bae8-454c-9f70-8b513dd0844e] succeeded in 0.0018890555948019028s: (10, 'The Fibonacci calculated with task id 4e2999a7-bae8-454c-9f70-8b513dd0844e was 55')
[2023-03-06 17:20:38,931: INFO/ForkPoolWorker-1] Task tasks.fibo_task[f639a24f-cbf5-403d-b243-4c5c54f5b77a] succeeded in 0.012269522994756699s: (6, 'The Fibonacci calculated with task id f639a24f-cbf5-403d-b243-4c5c54f5b77a was 8')
```

## 完整示例

`tasks.py`

```python
from math import sqrt
from celery import Celery

app = Celery('tasks', broker='redis://localhost/0', backend='redis://localhost/0')
# app.config.CELERY_RESULT_BACKEND = 'redis://192.168.99.89:6379/0'


@app.task
def sqrt_task(value):
    return sqrt(value)

@app.task
def fibo_task(value):
    a, b = 0,1
    for item in range(value):
        a, b = b, a + b
    message = "The Fibonacci calculated with task id %s was %d" % (fibo_task.request.id, a)
    return (value, message)
```

`tasks_dispatcher.py`

```python
import logging
from celery import Celery
from celery.result import AsyncResult
from typing import Dict

logger = logging.getLogger()
logger.setLevel(logging.DEBUG)
formatter = logging.Formatter('%(asctime)s - %(message)s')

ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)
ch.setFormatter(formatter)
logger.addHandler(ch)

app = Celery('tasks', broker='redis://localhost/0', backend='redis://localhost/0')

def manage_sqrt_task(value):
    result = app.send_task('tasks.sqrt_task', args=(value,))
    logger.info(result.get())



def manage_fibo_task(value_list):
    async_result_dict: Dict[int, AsyncResult] = {x: app.send_task('tasks.fibo_task',args=(x,)) for x in value_list}

    for key, value in async_result_dict.items():
        if value.ready():
            logger.info("Value [%d] -> %s" % (key, value.get()[1]))
        else:
            logger.info("Task [%s] is not ready" % value.task_id)

if __name__ == '__main__':
    input_list = [4, 3, 8, 6, 10]
    # print(manage_sqrt_task(4))
    print(manage_fibo_task(input_list))
```
