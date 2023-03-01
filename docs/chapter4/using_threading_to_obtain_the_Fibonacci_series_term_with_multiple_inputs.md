# 使用多线程解决斐波那契序列多输入问题

现在是时候实现了。任务是在给定多个输入值时并行执行斐波那契数列的各项。 出于教学目的，我们将固定四个元素中的输入值和四个线程来处理每个元素，模拟 worker 和要执行的任务之间的完美对称。 该算法将按如下方式工作：

1. 首先使用一个列表存储四个待输入值，这些值将被放入对于线程来说互相锁定的数据结构。
2. 输入值被放入可被锁定的数据结构之后，负责处理斐波那契序列的线程将被告知可以被执行。这时，我们可以使用python线程的同步机制`Condition`模块（`Condition`模块对共享变量提供线程之间的同步操作），模块详情请参考：<http://docs.python.org/3/library/threading.html#threading.Condition>{target="_blank"}。
3. 当每个线程结束斐波那契序列的计算后，分别把结果存入一个字典。

接下来我们将列出代码，并且讲述其中有趣的地方：

代码开始处我们加入了对编码额外的支持，导入`logging`, `threading`和`queue`模块。此外，我们还定义了我们例子中用到主要数据结构。
一个字典，被命名为`fibo_dict`，将用来存储输入输出数据，输入数据为`key`，计算结果（输出数据）为值。
我们同样定义了一个队列对象，该对象中存储线程间的共享数据（包括读写）。
我们把该对象命名为`shared_queue`。最后我们定义一个列表模拟程序的四个输入值。代码如下：

```python
#coding: utf-8

import logging, threading

from queue import Queue

logger = logging.getLogger()
logger.setLevel(logging.DEBUG)
formatter = logging.Formatter('%(asctime)s - %(message)s')

ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)
ch.setFormatter(formatter)
logger.addHandler(ch)

fibo_dict = {}
shared_queue = Queue()
input_list = [3, 10, 5, 7]
```

!!! info "下载示例代码"

    您可以从<http://www.packtpub.com>{target="_blank"}上的帐户下载您购买的所有 Packt 书籍的示例代码文件。 如果您在其他地方购买了本书，您可以访问 <http://www.packtpub.com/support>{target="_blank"} 并注册以便将文件直接通过电子邮件发送给您。

在下面的代码行中，我们将从名为 `Condition` 的线程模块中定义一个对象。 该对象旨在根据特定条件同步对资源的访问。

```python
queue_condition = threading.Condition()
```

使用 Condition 对象的用于控制队列的创建和在其中进行条件处理。

下一段代码是定义将由多个线程执行的函数。我们将称它为 `fibonacci_task`。`fibonacci_task`函数接收`condition`对象作为参数，它将控制`fibonacci_task`对`share_queue`的访问。在这个函数中，我们使用了`with`语句（关于`with`语句的更多信息，请参考<http://docs.python.org/3/reference/compound_stmts.html#with>）来简化内容的管理。如果没有`with`语句，我们将不得不明确地获取锁并释放它。有了`with`语句，我们可以在开始时获取锁，并在内部块的退出时释放它。`fibonacci_task`函数的下一步是进行逻辑评估，告诉当前线程："虽然`shared_queue`是空的，但要等待。" 这就是`condition`对象的`wait()`方法的主要用途。线程将等待，直到它得到通知说`shared_queue`可以自由处理。一旦我们的条件得到满足，当前线程将在`shared_queue`中获得一个元素，它马上计算斐波那契数列的值，并在`fibo_dict`字典中生成一个条目。最后，我们调用`task_done()`方法，目的是告知某个队列的任务已经被提取并执行。代码如下：

```python

def fibonacci_task(condition):
    with condition:
        while shared_queue.empty():
            logger.info("[%s] - waiting for elements in queue.." % threading.current_thread().name)
            condition.wait()
        else:
            value = shared_queue.get()
            a, b = 0, 1
            for item in range(value):
                a, b = b, a + b
                fibo_dict[value] = a
            shared_queue.task_done()
            logger.debug("[%s] fibonacci of key [%d] with result [%d]" % (threading.current_thread().name, value, fibo_dict[value]))
```

我们定义的第二个函数是`queue_task`函数，它将由负责为`shared_queue`填充要处理的元素的线程执行。我们可以注意到获取`condition`作为访问`shared_queue`的一个参数。对于`input_list`中的每一个项目，线程都会将它们插入`shared_queue`中。

将所有元素插入 `shared_queue` 后，该函数通知负责计算斐波那契数列的线程队列已准备好使用。 这是通过使用 `condition.notifyAll()` 完成的，如下所示：

```python
def queue_task(condition):
    logging.debug('Starting queue_task...')
    with condition:
        for item in input_list:
            shared_queue.put(item)
            logging.debug("Notifying fibonacci_task threadsthat the queue is ready to consume..")
            condition.notifyAll()  # python3.10 中使用 notify_all()
```

在下一段代码中，我们创建了一组包含四个线程的集合，它们将等待来自 `shared_queue` 的准备条件。 然后我们强调了线程类的构造函数，它允许我们定义函数。该线程将使用目标参数执行，该函数在`args`中接收的参数如下:

```python
threads = [
    threading.Thread(daemon=True, target=fibonacci_task,args=(queue_condition,)) 
    for i in range(4)
]
```

接着我们使用`thread`对象的`start`方法开始线程：

```python
[thread.start() for thread in threads]
```

然后我们创建一个线程处理`shared_queue`，然后执行该线程。代码如下：

```python
prod = threading.Thread(name='queue_task_thread', daemon=True, target=queue_task, args=(queue_condition,))
prod.start()
```

最后，我们对所有计算斐波那契数列的线程调用了`join()`方法。这个调用的目的是让让主线程等待子线程的调用，直到所有子线程执行完毕之后才结束主线程。请参考下面的代码：

```python
[thread.join() for thread in threads]
```

程序的执行结果如下：

```shell
$ python temp.py
2023-03-01 12:19:26,873 - [Thread-1 (fibonacci_task)] - waiting for elements in queue..
2023-03-01 12:19:26,873 - [Thread-2 (fibonacci_task)] - waiting for elements in queue..
2023-03-01 12:19:26,874 - [Thread-3 (fibonacci_task)] - waiting for elements in queue..
2023-03-01 12:19:26,874 - [Thread-4 (fibonacci_task)] - waiting for elements in queue..
2023-03-01 12:19:26,874 - Starting queue_task...
2023-03-01 12:19:26,874 - Notifying fibonacci_task threadsthat the queue is ready to consume..
2023-03-01 12:19:26,874 - Notifying fibonacci_task threadsthat the queue is ready to consume..
2023-03-01 12:19:26,874 - Notifying fibonacci_task threadsthat the queue is ready to consume..
2023-03-01 12:19:26,874 - Notifying fibonacci_task threadsthat the queue is ready to consume..
2023-03-01 12:19:26,875 - [Thread-1 (fibonacci_task)] fibonacci of key [3] with result [2]
2023-03-01 12:19:26,875 - [Thread-2 (fibonacci_task)] fibonacci of key [10] with result [55]
2023-03-01 12:19:26,875 - [Thread-4 (fibonacci_task)] fibonacci of key [5] with result [5]
2023-03-01 12:19:26,875 - [Thread-3 (fibonacci_task)] fibonacci of key [7] with result [13]
```

请注意，首先创建并初始化 `fibonacci_task` 线程，然后它们进入等待状态。 同时，创建 `queue_task` 并填充 `shared_queue`。 最后，`queue_task` 通知 `fibonacci_task` 线程它们可以执行它们的任务。

请注意，`fibonacci_task` 线程的执行不遵循顺序逻辑，每次执行的顺序可能不同。 这就是使用线程的一个特点：**非确定性**(non-determinism)。

## 完整例子

译者注:

```python

#coding: utf-8

import logging, threading

from queue import Queue

logger = logging.getLogger()
logger.setLevel(logging.DEBUG)
formatter = logging.Formatter('%(asctime)s - %(message)s')

ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)
ch.setFormatter(formatter)
logger.addHandler(ch)

fibo_dict = {}
shared_queue = Queue()
input_list = [3, 10, 5, 7]

queue_condition = threading.Condition()

def fibonacci_task(condition):
    with condition:
        while shared_queue.empty():
            logger.info("[%s] - waiting for elements in queue.." % threading.current_thread().name)
            condition.wait()
        else:
            value = shared_queue.get()
            a, b = 0, 1
            for item in range(value):
                a, b = b, a + b
                fibo_dict[value] = a 
            shared_queue.task_done()
            logger.debug("[%s] fibonacci of key [%d] with result [%d]" % (threading.current_thread().name, value, fibo_dict[value]))

def queue_task(condition):
    logging.debug('Starting queue_task...')
    with condition:
        for item in input_list:
            shared_queue.put(item)
            logging.debug("Notifying fibonacci_task threadsthat the queue is ready to consume..")
            condition.notify_all()




if __name__ == "__main__":
    threads = [
        threading.Thread(daemon=True, target=fibonacci_task,args=(queue_condition,)) 
        for i in range(4)
    ]

    [thread.start() for thread in threads]

    prod = threading.Thread(name='queue_task_thread', daemon=True, target=queue_task, args=(queue_condition,))
    prod.start()

    [thread.join() for thread in threads]
```
