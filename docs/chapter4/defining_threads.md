# 什么是线程

**线程是进程中的不同执行线**。 让我们把一个程序想象成一个蜂巢，在这个蜂巢内部有一个收集花粉的过程。 这个采集过程是由几只工蜂同时工作来完成的，以解决花粉不足的问题。 工蜂扮演着线程的角色，在进程内部活动并共享资源来执行它们的任务。

线程属于同一个进程，共享同一个内存空间。 因此，开发人员的任务是控制和访问这些内存区域。

## 使用线程的优点和缺点

在决定使用线程时必须考虑一些**优点**和**缺点**，这取决于用于实现解决方案的语言和操作系统。

使用线程的**优势**如下所示:

- 同一进程内的线程**通信**、**数据定位**、**共享信息**的速度快
- 线程的创建比进程的创建成本更低，因为不需要复制主进程上下文中包含的所有信息
- 通过处理器的高速缓存优化内存访问，充分利用**数据局部性**(data locality)的优势。

使用线程的**缺点**如下:

- 数据共享允许快速通信。 但是，它也允许缺乏经验的开发人员引入难以解决的错误。
- 数据共享限制了解决方案的灵活性。 例如，迁移到分布式架构可能会让人头疼。 通常，它们限制了算法的可扩展性。

!!! info ""

    在 Python 编程语言中，由于 GIL，使用**计算密集型**(CPU-bound)的线程可能会损害应用程序的性能。

## 理解不同类型的线程

有两种类型的线程，**内核线程**和**用户线程**。 **内核线程是由操作系统创建和管理的线程**, 其上**下文的交换**、**调度**和**结束**都由当前操作系统的内核来进行管理。 对于**用户线程**，这些状态由**包**(package)开发人员控制。

我们可以引用每种线程的一些优点:

<table>
<thead>
    <tr>
        <td style="width:95px;">线程类型</td>
        <td style="text-align:center;">优点</td>
        <td style="text-align:center;">缺点</td>
    </tr>
</thead>
<tbody>
    <tr>
        <td style="vertical-align:middle;">内核线程</td>
        <td style="vertical-align:middle;">
            一个内核线程其实就是一个进程. 因此即使一个内核线程被阻塞了,其他的内核线程仍然可以运行。<br/>
            内核线程可以在不同的 CPU 上运行。
        </td>
        <td style="vertical-align:middle;">
            创建线程和线程间同步的消耗太大<br/>
            实现依赖于平台
        </td>
    </tr>
    <tr>
        <td style="vertical-align:middle;">用户线程</td>
        <td style="vertical-align:middle;">
            用户线程的创建和线程间同步的开销较少<br/>
            用户线程是平台无关的。
        </td>
        <td style="vertical-align:middle;">
            同一进程中的所有用户线程都对应一个内核线程. 因此,若该内核线程被阻塞,则所有相应的用户线程都会被阻塞。<br/>
            不同用户线程无法运行在不同CPU上
        </td>
    </tr>
</tbody>
</table>

## 线程的状态

线程的生命周期有五种可能的状态。它们如下：

- **新建**(Creation): 该过程的主要动作就是创建一个新线程, 创建完新线程后,该线程被发送到待执行的线程队列中。
- **运行**(Execution): 该状态下,线程获取到并消耗CPU资源。
- **就绪**(Ready): 该状态下,线程在待执行的线程队列中排队,等待被执行
- **阻塞**(Blocked): 该状态下,线程由于等待某个事件(例如I/O操作)的出现而被阻塞. 这时线程并不使用CPU。
- **死亡**(Concluded): 该状态下,线程释放执行时使用的资源并结束整个线程的生命周期。

## 是使用threading模块还是_thread模块

Python提供了两个模块来实现基于系统的线程: **`_thread`模块**(该模块提供了使用线程相关的较低层的API; 它的文档可以在<http://docs.python.org/3.3/library/_thread.html> 找到)和**`threading`模块**(该模块提供了使用线程相关的更高级别的API; 它的文档可以在 <http://docs.python.org/3.3/library/threading.html> 中找到). **`threading`模块**提供的接口要比**`_thread`模块**的结构更友好一些. 至于具体选择哪个模块取决于开发者, 如果开发人员发现在较低级别使用线程很容易，实现他们自己的线程池并拥抱锁和其他原始功能(features)，他/她宁愿使用`_thread`。 否则，`threading`是最明智的选择。