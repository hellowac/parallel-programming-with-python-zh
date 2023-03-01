# 理解进程的概念

我们必须将操作系统中的**进程**(processes)理解为执行中的程序及其资源的**容器**。 与执行中的程序有关的所有内容都可以通过它所代表的进程进行管理——它的**数据区域**(data area)、它的**子进程**(child processes)、它的**资产**(estates)以及它**与其他进程的通信**(communication with other processes)。

## 理解进程模型

**进程**(processes)具有相关的信息和资源，可以对其进行操作和控制。 操作系统有一个称为**进程控制块** (PCB) 的结构，它存储有关进程的信息。 例如，PCB 可能存储以下信息：

- **Process ID**(PID): 这是唯一的整数值（无符号），用于标识操作系统中的进程。
- **程序计数器**(Program counter): 这包含要执行的下一条程序指令的地址。
- **I/O信息**(I/O information): 这是与进程相关联的打开文件和设备的列表
- **内存分配**(Memory allocation): 这存储有关进程使用和保留的内存空间以及分页表的信息。
- **CPU调度**(CPU scheduling): 这存储有关进程优先级的信息并指向**交错队列**。(staggering queues)。
- **优先级**(Priority:): 这定义了进程在获取 CPU 时的优先级
- **当前状态**(Current state): 这表明进程是就绪(ready)、等待(waiting)还是正在运行(running)
- **CPU 注册表**(CPU registry): 这存储堆栈指针和其他信息。

### 定义进程状态

**进程**(processes)拥有跨越其生命周期的三种状态； 它们如下：

- **运行状态**(Running): 该进程正在使用 CPU。
- **就绪状态**(Ready): 在进程队列中等待的进程现在可以使用 CPU。
- **等待状态**（Waiting）: 进程正在等待与它执行的任务相关的一些 I/O 操作。
