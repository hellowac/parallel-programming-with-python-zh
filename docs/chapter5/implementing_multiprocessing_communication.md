# 实现多进程间通信

[multiprocessing](http://docs.python.org/3/library/multiprocessing.html)模块允许进程间以两种方式进行通信，这两种方式都基于消息传递机制。 如前所述，由于缺乏同步机制因此不得不采取消息传递机制，因此是数据副本在进程之间交换。

## 使用multiprocessing.Pipe模块

**管道**(pipe)由在两个端点（通信中的两个进程）之间建立通信的机制组成。 这是一种创建通道以便在进程之间交换消息的方法。

!!! warning ""

    Python 官方文档建议每两个端点使用一个管道，因为不能保证另一个端点同时读取安全。

为了举例说明`multiprocessing.Pipe`对象的使用，我们来实现一个创建两个进程A和B的Python程序，进程A发送一个1到10的随机整数值给进程B，进程B会显示它在屏幕上。 现在，让我们一步步介绍这个程序。

我们首先导入一些我们程序中需要的包，如下：

```python
import os, random
from multiprocessing import Process, Pipe
```

通过`os`模块的[os.getpid()](http://docs.python.org/3.3/library/os.html)方法使得我们可以获得进程的PID。[os.getpid()](http://docs.python.org/3.3/library/os.html)将以一种透明的方式返回程序的PID，在我们的示例中，`os.getpid()` 调用将以透明形式返回负责运行任务 `producer_task` 和 `consumer_task` 的各自进程的 `PID`。

在程序的下一部分，我们将定义`producer_task`函数，除此之外，该函数将使用`random.randint(1, 10)`调用生成一个随机数。这个函数的关键点被称为`conn.send(value)`，它使用`Pipe`在主程序的流量中生成的连接对象，该连接对象(conn)已被作为参数发送到该函数。观察`producer_task`函数的全部内容，如下所示:

```python
def producer_task(conn):
    value = random.randint(1, 10)
    conn.send(value)
    print('Value [%d] send by PID [%d]' % (value, os.getpid()))
    conn.close()
```

!!! warning ""

    永远不要忘记总是调用`Pipe`连接的`close()`方法，该连接通过发送方法发送数据。当不再使用时，这对于最终释放与通信通道相关的资源是很重要的。

消费者进程要执行的任务非常简单，它的唯一目标是在屏幕上打印接收到的值，接收消费者进程的`PID`。 为了从通信通道获取发送的值，我们使用了 [conn.recv()](http://docs.python.org/dev/library/multiprocessing.html#multiprocessing.Connection.recv){target="_blank"} 调用。 `consumer_task` 函数的实现最终如下所示：

```python
def consumer_task(conn):
    print('Value [%d] received by PID [%d]' % (conn.recv(), os.getpid()))
```

我们这个小例子的最后部分实现了对`Pipe()`对象的调用，创建了两个连接对象，将被消费者和生产者进程使用。在这个调用之后，生产者和消费者进程被创建，分别发送`consumer_task`和`producer_task`函数作为目标函数，我们可以在下面的完整代码中看到。

```python
if __name__ == '__main__':
    producer_conn, consumer_conn = Pipe()
    consumer = Process(target=consumer_task,args=(consumer_conn,))
    producer = Process(target=producer_task,args=(producer_conn,))
                                                                                
    consumer.start()
    producer.start()
                                                                                
    consumer.join()
    producer.join()
```

定义完进程后，就该调用`start()`方法来启动执行，并调用`join()`方法，这样主进程就会等待生产者和消费者进程的执行。

在下面的截图中，我们可以看到`multiprocessing_pipe.py`程序的输出。

```shell
$ python multiprocessing_pipe.py 
Value [6] send by PID [95980]
Value [6] received by PID [95979]
```

## 理解multiprocessing.Queue模块

在上一节中，我们分析了管道的概念，通过创建一个通信通道在进程之间建立通信。现在，我们将分析如何有效地建立这种通信，利用`Queue`对象，它在`multiprocessing`模块中实现。`multiprocessing.Queue`的可用接口与`queue.Queue`相当类似。然而，内部实现使用了不同的机制，比如使用了`thread`的内部线程`feeder` ，它将数据从队列的数据缓冲区传输到与目标进程相关的管道。管道和队列机制都利用了**消息传递机制**，这使用户无需使用**同步机制**，从而节省了使用**同步机制**带来的开销。

!!! warning ""

    虽然使用`multiprocessing.Queue`的用户不需要使用同步机制，例如`Locks`，但在内部，这些机制被用来在缓冲区和管道之间传输数据，以完成通信。

## 完整示例

译者注:

```python
import os, random
from multiprocessing import Process, Pipe

def producer_task(conn):
    value = random.randint(1, 10)
    conn.send(value)
    print('Value [%d] send by PID [%d]' % (value, os.getpid()))
    conn.close()


def consumer_task(conn):
    print('Value [%d] received by PID [%d]' % (conn.recv(), os.getpid()))

if __name__ == '__main__':
    producer_conn, consumer_conn = Pipe()
    consumer = Process(target=consumer_task,args=(consumer_conn,))
    producer = Process(target=producer_task,args=(producer_conn,))
                                                                                
    consumer.start()
    producer.start()
                                                                                
    consumer.join()
    producer.join()
```
