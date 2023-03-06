# 分发简单任务

在之前，我们已经建立好环境。下面测试一下环境，发送一个计算平方根的任务。

定义任务模块`tasks.py`。在开始，导入必须的模块。

```python
from math import sqrt
from celery import Celery
```

然后，创建`Celery`实例，代表客户端应用：

```python
app = Celery('tasks', broker='redis://192.168.25.21:6379/0')
```

在初始化时我们传入了模块的名称和`broker`的地址。

然后，启动`result backend`，如下：

```python
app.config.CELERY_RESULT_BACKEND = 'redis://192.168.25.21:6379/0'

# 较新的版本(v5.2.7)直接填充在celery app的初始化参数中.
app = Celery('tasks', broker='redis://localhost/0', backend='redis://localhost/0')
```

用`@app.tack`装饰器定义任务：

```python
@app.task
def sqrt_task(value):
    return sqrt(value)
```

到此，我们完成了`tasks.py`模块的定义，我们需要初始化服务端的`workers`。我们创建了一个单独的目录叫做`8397_07_broker`。拷贝`tasks.py`模块到这个目录，运行如下命令：

```shell
$celery –A tasks worker –-loglevel=INFO
```

上述命令初始化了**Clery Server**，`—A`代表`Celery`应用。下图是初始化的部分截图

```shell
$# celery -A tasks worker  --loglevel=INFO
/opt/celery_env/lib/python3.9/site-packages/celery/platforms.py:840: SecurityWarning: You're running the worker with superuser privileges: this is
absolutely not recommended!

Please specify a different user using the --uid option.

User information: uid=0 euid=0 gid=0 egid=0

  warnings.warn(SecurityWarning(ROOT_DISCOURAGED.format(

 -------------- celery@ch1.nauu.com v5.2.7 (dawn-chorus)
--- ***** -----
-- ******* ---- Linux-3.10.0-957.el7.x86_64-x86_64-with-glibc2.17 2023-03-06 16:12:10
- *** --- * ---
- ** ---------- [config]
- ** ---------- .> app:         tasks:0x7fe5cbea9b80
- ** ---------- .> transport:   redis://localhost:6379/0
- ** ---------- .> results:     redis://localhost/0
- *** --- * --- .> concurrency: 2 (prefork)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** -----
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery


[tasks]
  . tasks.square_root

[2023-03-06 16:12:10,866: INFO/MainProcess] Connected to redis://localhost:6379/0
[2023-03-06 16:12:10,871: INFO/MainProcess] mingle: searching for neighbors
[2023-03-06 16:12:11,897: INFO/MainProcess] mingle: all alone
[2023-03-06 16:12:11,929: INFO/MainProcess] celery@ch1.nauu.com ready.
```

现在，**Celery Server**等待接收任务并且发送给`workers`。

下一步就是在客户端创建应用调用`tasks`。

!!! info ""

    上述步骤不能忽略，因为下面会用在之前创建的东西。

在客户端机器，我们有**celery_env**虚拟环境，现在创建一个`task_dispatcher.py`模块很简单，如下步骤；

1. 导入logging模块来显示程序执行信息，导入Celery模块：

    ```python
    import logging
    from celery import Celery
    ```

2. 下一步是创建Celery实例，和服务端一样：

    ```python
    #logger configuration...
    app = Celery('tasks', broker='redis://192.168.25.21:6379/0')
    app.conf.CELERY_RESULT_BACKEND = 'redis://192.168.25.21:6397/0'
    ```

由于我们在接下的内容中要复用这个模块来实现任务的调用，下面我们创建一个方法来封装`sqrt_task(value)`的发送，我们将创建`manage_sqrt_task(value)`方法：

```python
def manage_sqrt_task(value):
    result = app.send_task('tasks.sqrt_task', args=(value,))
    logging.info(result.get())
```

从上述代码我们发现客户端应用不需要知道服务端的实现。通过**Celery**类中的`send_task`方法，我们传入`module.task`格式的字符串和以元组的方式传入参数就可以调用一个任务。最后，我们看一看`log`中的结果。
在`__main__`中，我们调用了`manage_sqrt_task(value)`方法：

```python
if __name__ == '__main__':
    manage_sqrt_task(4)
```

下面的截图是执行`task_dispatcher.py`文件的结果：

```shell
[2023-03-06 16:18:45,481: INFO/MainProcess] Task tasks.sqrt_task[3ecab729-f1cb-4f29-bb47-b713b2e563ed] received
[2023-03-06 16:18:45,500: INFO/ForkPoolWorker-2] Task tasks.sqrt_task[3ecab729-f1cb-4f29-bb47-b713b2e563ed] succeeded in 0.015412827953696251s: 2.0
```

在客户端，通过`get()`方法得到结果，这是通过`send_task()`返回的`AsyncResult`实例中的重要特征。结果如下图：

```shell
$# python task_dispatcher.py
2023-03-06 16:26:05,841 - 2.0
```

## 完整案例

`tasks.py`

```python
from math import sqrt
from celery import Celery

app = Celery('tasks', broker='redis://localhost/0', backend='redis://localhost/0')


@app.task
def sqrt_task(value):
    return sqrt(value)
```

`task_dispatcher.py`

```python
import logging
from celery import Celery

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


if __name__ == '__main__':
    print(manage_sqrt_task(4))
```
