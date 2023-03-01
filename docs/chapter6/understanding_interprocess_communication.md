# 理解进程间通讯

进程间通信 (Interprocess communication - IPC) 由允许在进程之间交换信息的机制组成。

有多种实现 IPC 的方法，通常，它们取决于为运行时环境选择的体系结构。 在某些情况下，例如，进程在同一台机器上运行，我们可以使用各种类型的通信，例如**共享内存**、**消息队列**和**管道**。 例如，当进程物理分布在集群中时，我们可以使用**套接字**(sockets)和**远程过程调用** (Remote Procedure Call - RPC)。

在第 5 章，使用 `Multiprocessing` 和 `ProcessPoolExecutor`，我们验证了常规管道的使用等。 我们还研究了具有共同父进程的进程之间的通信。 但是，有时需要在不相关的进程（具有不同父进程的进程）之间进行通信。 我们可能会问自己，不相关进程之间的通信是否可以通过它们的寻址空间来完成。 尽管如此，**一个进程永远不会从另一个进程访问寻址空间**。 因此，我们必须使用称为**命名管道**的机制。

## 探索命名管道

在 **POSIX** 系统中，例如 **Linux**，我们应该记住一切，绝对一切，都可以总结为文件。对于我们执行的每个任务，在某处都有一个文件，我们还可以找到一个附加到它的文件描述符，它允许我们操作这些文件。

!!! info "文件描述符"

    文件描述符是允许用户程序访问文件以进行读/写操作的机制。 通常，文件由唯一的文件描述符引用。 有关文件描述符的更多信息，请访问<https://www.ibm.com/docs/en/aix/7.2?topic=volumes-using-file-descriptors>{target="_blank"}(原文地址为: <http://publib.boulder.ibm.com/infocenter/pseries/v5r3/index.jsp?topic=/com.ibm.aix.genprogc/doc/genprogc/fdescript.html>,但找不到了，找到了一个替代描述文件描述符的。)

**命名管道**(Named pipes)不过是允许通过使用与特殊文件相关联的文件描述符进行 IPC 通信的机制，例如，用于写入和读取数据的**先进先出** (FIFO) 方案。 命名管道与常规管道的不同之处在于它们管理信息的方法。 **命名管道**(Named pipes)使用文件系统中的文件描述符和特殊文件，而常规管道是在内存中创建的。

## 在python中使用命名管道

**命名管道**在 Python 中的使用非常简单，我们将通过实现两个执行单向通信的程序来说明这一点。 第一个程序名为`write_to_named_pipe.py`，其功能是在管道中写入一条22字节的消息，通知一个字符串和生成它的进程的PID。 第二个程序称为 `read_from_named_pipe.py`，它将读取消息并显示消息内容，添加其 PID。

在执行结束时，`read_from_named_pipe.py` 进程将显示一条形如"I pid [\<The PID of reader process\>] received a message => Hello from pid [the PID of writer process"的消息。

为了说明在命名管道中写入和读取进程之间的相互依赖性，我们将在两个不同的控制台中执行读取和写入。 但在此之前，让我们分析一下这两个程序的代码。

### 往命名管道写入数据

在 Python 中，命名管道是通过系统调用实现的。 在下面的代码中，我们将逐行解释 `write_to_named_pipe.py` 程序中代码的功能。

我们从 `os` 模块的输入开始，它将提供对系统调用的访问，我们将使用以下代码行：

```python
import os
```

接下来我们会解释__main__代码块,在该代码块中创建了命名管道以及一个用于存储消息的FIFO的特殊文件. __main__代码块中的第一行代码定义了命名管道的标签.

```python
named_pipe = "my_pipe"
```

接下来我们检查该命名管道是否已经存在,若不存在,则调用`mkfifo`系统调用来创建这个命名管道.

```python
if not os.path.exists(named_pipe):
    os.mkfifo(named_pipe)
```

`mkfifo`调用会创建一个特殊的文件,该文件对通过命名管道读写的消息实现了**FIFO**机制.

我们再以一个命名管道和一个行如"Hello from pid [%d]"的消息来作为参数调用函数`write_message`. 该函数会将消息写入到(作为参数传递给它的)命名管道所代表的文件中. `write_message`函数定义如下:

```python
def write_message(input_pipe, message):
    fd = os.open(input_pipe, os.O_WRONLY)
    os.write(fd, (message % str(os.getpid())))
    os.close(fd)
```

我们可以观察到,在函数的第一行,我们调用一个系统调用:`open`. 该系统调用若成功的话会返回一个文件描述符, 通过该文件描述符我们就能够读写那个`FIFO`文件中的数据. 请注意,我们可以通过`flags`参数控制打开**FIFO**文件的模式. 由于`write_message`函数紧紧需要写数据,因此我们使用如下代码:

```python
fd = os.open(input_pipe, os.O_WRONLY)
```

在成功打开命名管道后,我们使用下面代码写入消息:

```python
os.write(fd, (message % os.getpid()))
```

最后,请一定记着使用`close`关闭通讯渠道,这样才能释放被占用的计算机资源.

```python
os.close(fd)
```

### 从命名管道读取数据

我们实现`read_from_pipe.py`来读取命名管道. 当然,改程序也需要借助`os`模块才能操作命名管道. 改程序的主要代码很简单: 首先,我们定义了所使用命名管道的标签,该标签需要与写进程所用的命名管道同名.

```python
named_pipe = "my_pipe"
```

然后,我们调用`read_message`函数,该函数会读取`write_to_named_pipe.py`写入的内容. `read_message`函数的定义如下:

```python
# 此处原文应该有错

def read_message(input_pipe):
    fd = os.open(input_pipe, os.O_RDONLY)
    message = "I pid [%d] received a message => %s" % (os.getpid(), os.read(fd, 22))
    os.close(fd)
    return message
```

`open`调用不需要再介绍。 这里的新事物是我们的读取调用，它以字节为单位执行数量的读取。 在我们的例子中，如果给出了文件描述符，它就是 `22` 个字节。 消息被读取后，由函数返回。 最后，必须执行`close`调用以关闭通信通道。

!!! info ""

    要验证已打开文件描述符的有效性。需要由用户处理在使用文件描述符和命名管道时产生的相关异常。

最终,下面的截屏显示了`write_to_named_pip`和`read_from_named_pipe`程序的执行结果.

```shell
>$ python write_to_named_pipe.py

```

```shell
>$ python read_from_pipe.py
I pid [61032] received a message => Hello 61017
```

## 完整示例

译者注:

```python
# write_to_named_pip.py

import os
import sys


def write_message(input_pipe, message):
    fd = os.open(input_pipe, os.O_WRONLY)
    os.write(fd, (message % str(os.getpid())).encode()) # 管道通信为字节，这里需要转码
    os.close(fd)


if __name__ == "__main__":
    named_pipe = "my_pipe"

    if not os.path.exists(named_pipe):
        os.mkfifo(named_pipe)

    write_message(named_pipe, "Hello %s")

```

```python
# read_from_named_pipe.py

import os
import sys


def read_message(input_pipe):
    fd = os.open(input_pipe, os.O_RDONLY)
    message = "I pid [%d] received a message => %s" % (
        os.getpid(),
        os.read(fd, 22).decode(),  # 管道通信为字节，这里需要转码
    )
    os.close(fd)
    return message


if __name__ == "__main__":
    named_pipe = "my_pipe"

    if not os.path.exists(named_pipe):
        os.mkfifo(named_pipe)

    print(read_message(named_pipe))

```
