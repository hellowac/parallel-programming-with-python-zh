# 使用多进程解决斐波那契序列多输入问题

下面我们将使用多进程解决多输入情况下的斐波那契数列问题，而不是之前我们使用的多线程的方法。

`multiprocessing_fibonacci.py` 代码使用了 `multiprocessing` 模块，为了运行，它导入了一些基本模块，我们可以在以下代码中观察到：

```python
import sys, time, random, re, requests
import concurrent.futures
from multiprocessing import cpu_count, current_process, Manager
```

前面的章节中已经提到了一些导入； 尽管如此，以下某些导入确实值得特别注意：

- **cpu_count**: 这是一个允许获取机器中 CPU 数量的函数。
- **current_process**: 这是一个允许获取有关当前进程的信息的函数，例如，它的名称。
- **Manager**: 这是一种允许通过代理在不同进程之间共享 `Python` 对象的类型。（更多信息参考:<http://docs.python.org/3/library/multiprocessing.html#multiprocessing.Manager>{target="_blank"}）

按照代码，我们可以注意到第一个函数的行为有所不同； 它将在 0-14 次迭代期间以 1 到 20 的间隔生成随机值。 这些值将作为键插入到 `fibo_dict` 中，这是一个由 `Manager` 对象生成的字典。

!!! warning ""

    使用消息传递的方法更为常见。 然而，在某些情况下，我们需要在不同进程之间共享一段数据，正如我们在 `fibo_dict` 字典中看到的那样。

接下来让我们一起来看`producer_task`方法，如下：

```python
def producer_task(q, fibo_dict):
    for i in range(15):
        value = random.randint(1, 20)
        fibo_dict[value] = None

        print("Producer [%s] putting value [%d] into queue.." % (current_process().name, value))
        q.put(value)
```

下一步是定义函数，该函数将为 `fibo_dict` 中的每个键计算斐波那契数列值。值得注意的是，与上一章中介绍的函数相关的唯一区别是使用 `fibo_dict` 作为参数以允许在不同进程使用它。

下面是`consumer_task`方法，如下：

```python
def consumer_task(q, fibo_dict):
    while not q.empty():
        value = q.get(True, 0.05)
        a, b = 0, 1
        for item in range(value):
            a, b = b, a+b
        fibo_dict[value] = a
        print("consumer [%s] getting value [%d] from queue..." % (current_process().name, value))
```

为了进一步了解代码，我们看看程序的主代码块。在这个主代码块中，定义了以下一些变量：

- **data_queue**: 该参数由`multiprocessing.Queueu`来创建，是进程安全的
- **number_of_cpus**: 该参数由`multiprocessing.cpu_count`方法获得，获得机器cpu的个数
- **fibo_dict**: 这个字典类型变量从`Manager`实例获得，保存多进程计算结果

在代码中，我们创建了一个名为 `producer` 的进程，以使用 `producer_task` 函数使用随机值填充 `data_queue`，如下所示：

```python
producer = Process(target=producer_task, args=(data_queue, fibo_dict))
producer.start()
producer.join()
```

我们可以注意到`Process`实例的初始化过程和我们之前的`Thread`实例初始化过程类似。初始化函数接收`target`参数作为进程中要执行的函数，和`args`参数作为`target`传入的函数的参数。接下来我们通过`start()`函数开始进程，然后使用`join()`方法，等待`producer`进程执行完毕。

在下一个块中，我们定义了一个名为 `consumer_list` 的列表，它将存储已初始化进程的消费者列表。 创建此列表的原因是仅在所有 `worker` 的进程开始后调用 `join()`。 如果为循环中的每个项目调用 `join()` 函数，那么只有第一个 `worker` 会执行该作业，因为下一次迭代将被阻塞，等待当前 `worker` 结束，最后下一个`worker`将没有其他要处理的内容； 以下代码代表了这种情况：

```python
consumer_list = []
number_of_cpus = cpu_count()

for i in range(number_of_cpus):
    consumer = Process(target=consumer_task, args=(data_queue, fibo_dict))
    consumer.start()
    consumer_list.append(consumer)

[consumer.join() for consumer in consumer_list]
```

最终，我们在 `fibo_dict` 中展示了迭代的结果，如下截图所示：

```shell
$ python multiprocessing_fibonacci.py
Producer [Process-2] putting value [8] into queue..
Producer [Process-2] putting value [10] into queue..
Producer [Process-2] putting value [19] into queue..
Producer [Process-2] putting value [6] into queue..
Producer [Process-2] putting value [17] into queue..
Producer [Process-2] putting value [18] into queue..
Producer [Process-2] putting value [19] into queue..
Producer [Process-2] putting value [17] into queue..
Producer [Process-2] putting value [18] into queue..
Producer [Process-2] putting value [4] into queue..
Producer [Process-2] putting value [6] into queue..
Producer [Process-2] putting value [7] into queue..
Producer [Process-2] putting value [9] into queue..
Producer [Process-2] putting value [4] into queue..
Producer [Process-2] putting value [19] into queue..
consumer [Process-4] getting value [8] from queue...
consumer [Process-6] getting value [6] from queue...
consumer [Process-9] getting value [19] from queue...
consumer [Process-13] getting value [6] from queue...
consumer [Process-11] getting value [4] from queue...
consumer [Process-4] getting value [9] from queue...
consumer [Process-8] getting value [18] from queue...
consumer [Process-10] getting value [18] from queue...
consumer [Process-3] getting value [19] from queue...
consumer [Process-5] getting value [10] from queue...
consumer [Process-6] getting value [4] from queue...
consumer [Process-7] getting value [17] from queue...
consumer [Process-14] getting value [17] from queue...
consumer [Process-12] getting value [7] from queue...
consumer [Process-9] getting value [19] from queue...
{8: 21, 10: 55, 19: 4181, 6: 8, 17: 1597, 18: 2584, 4: 3, 7: 13, 9: 34}
```

## 完整示例

译者注:

```python
#coding: utf-8
import sys, time, random
import concurrent.futures
from multiprocessing import cpu_count, current_process, Manager, Process, Queue

def producer_task(q, fibo_dict):
    for i in range(15):
        value = random.randint(1, 20)
        fibo_dict[value] = None
        print("Producer [%s] putting value [%d] into queue.." % (current_process().name, value))
        q.put(value)

def consumer_task(q, fibo_dict):
    while not q.empty():
        value = q.get(True, 0.05)
        a, b = 0, 1
        for item in range(value):
            a, b = b, a+b
        fibo_dict[value] = a
        time.sleep(random.randint(1, 3))  # 由于现代计算机cpu处理太快，这里随机sleep几秒
        print("consumer [%s] getting value [%d] from queue..." % (current_process().name, value))

if __name__ == "__main__":
    fibo_dict = Manager().dict()  # 如果替换为 {}, 则没有共享对象的功能，打印出来将是空的。
    data_queue = Queue()

    producer = Process(target=producer_task, args=(data_queue, fibo_dict))
    producer.start()
    producer.join()

    consumer_list = []
    number_of_cpus = cpu_count()
    for i in range(number_of_cpus):
        consumer = Process(target=consumer_task, args=(data_queue, fibo_dict))
        consumer.start()
        consumer_list.append(consumer)

    [consumer.join() for consumer in consumer_list]

    print(fibo_dict)

```
