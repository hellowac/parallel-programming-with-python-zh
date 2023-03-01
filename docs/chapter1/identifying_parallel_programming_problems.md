# 识别并行编程的问题

勇敢的键盘战士在并行编程幽灵居住的土地上作战时可能会遇到一些经典问题。 当没有经验的程序员使用结合了共享状态的 worker 时，许多这些问题会更频繁地发生。 其中一些问题将在以下各节中进行描述。

## 死锁

**死锁**(Deadlock)是指两个或多个 worker 无限期地等待资源释放的情况，由于某种原因，该资源被同一组的 worker 阻塞。 为了更好地理解，我们将使用另一个真实案例。 想象一下入口处有一扇旋转门的银行。 客户 A 转向一侧，这将允许他进入银行，而客户 B 试图通过这个旋转门的入口侧离开银行，这样两个客户都会被困在推门处，但无处可去。 这种情况在现实生活中会很滑稽，但在编程中会很悲惨。

!!! info ""

    **死锁**(Deadlock)是一种现象，其中进程等待释放任务条件的发生，但这种情况永远不会发生

## 饥饿

这个问题是由于一个或者多个进程不公平的竞争所引起的副作用，这会花费更多的时间来执行任务。想象有一组进程，A进程正在执行繁重的任务，而且这个任务还有数据处理优先级。现在，想象一下，高优先级的进程A持续不断的占用CPU，而低优先级的进程B将永远没有机会。因此可以说进程B在CPU周期中是饥饿的。

> **饥饿**（Starvation）是由于进程排名策略调整不当造成的。

## 竞态条件

当一个进程的结果取决于执行结果的顺序，而这个顺序由于缺乏同步机制而被打破时，我们就会面临竞态条件。 它们是由在大型系统中极难过滤的问题引起的。 例如，一对夫妇有一个联名账户； 操作前的初始余额为 100 美元。 下表显示了常规情况，其中有保护机制和预期事实的顺序，以及结果：

!!! info "常规操作而不会出现静态条件"

    | 丈夫     | 妻子     | 账户余额（美元） |
    | -------- | -------- | ---------------- |
    |          |          | 100              |
    | 读取余额 |          | 100              |
    | 存款20   |          | 100              |
    | 结束操作 |          | 120              |
    |          | 读取余额 | 120              |
    |          | 取款10   | 120              |
    |          | 结束操作 | 110              |

在下表中，有问题的场景出现了。假设账户没有同步机制，并且操作的顺序也和预期不一样。

!!! info "类比在联合账户和竞争条件下平衡问题"

    | 丈夫                  | 妻子                  | 账户余额（美元） |
    | --------------------- | --------------------- | ---------------- |
    |                       |                       | 100              |
    | 读取余额              |                       | 100              |
    | 取款100               |                       | 100              |
    |                       | 读取余额              | 100              |
    |                       | 取款10                | 100              |
    | 结束操作<br/>更新余额 |                       | 0                |
    |                       | 结束操作<br/>更新余额 | 90               |

由于在操作的顺序中意外的缺乏同步，最终结果存在明显的不一致。并行编程的特征之一是不确定性。无法预见两个 worker 将在什么时候运行，甚至谁先运行。 因此，同步机制必不可少。

!!! info ""

    **不确定性**(Non-determinism)，如果与缺乏同步机制相结合，可能会导致竞争条件问题