# 并行、并发以及分布式编程的对比分析

并行编程可以被定义为一种模型，这个模型旨在创造一种能与**被准备用于同时执行代码指令的环境相兼容**的程序。并行技术被用于软件开发还不是很长。几年前，处理器在其他组件中只有一个**算术逻辑单元** (**ALU**)，它在一个时间空间内一次只能执行一条指令。 多年来，只考虑了一个以**赫兹**(Hz)为单位的时钟，以确定处理器在给定时间间隔内可以处理的指令数。 时钟数量越多，就 **KHz**（每秒千次操作）、**MHz**（每秒百万次操作）和当前的 **GHz**（每秒十亿次操作）而言，可能执行的指令越多。

总而言之，每个周期提供给处理器的指令越多，执行速度就越快。 在80 年代，*Intel 80386* 出现了革命性的处理器，它允许以先发制人的方式执行任务，也就是说，可以定期中断一个程序的执行，为另一个程序提供处理器时间； 这意味着基于*时间分片*(time-slicing)的**伪并行**(pseudo-parallelism)。

在 80 年代后期，*Intel 80486* 实现了流水线系统(pipelining system)，实际上将执行阶段划分为不同的子阶段。 实际上，在处理器的一个周期中，我们可以在每个子阶段同时执行不同的指令。

上一节中提到的所有进步都导致了性能的多项改进，但这还不够，因为我们面临着一个微妙的问题，最终会成为所谓的**摩尔定律** (<http://www.mooreslaw.org>{target="_blank"}).

探寻满足时钟高能耗的过程最终与物理限制发生冲突； 处理器会消耗更多的能量，从而产生更多的热量。 此外，还有另一个同样重要的问题：便携式计算机市场在 20 世纪 90 年代加速发展。 因此，拥有能够使这些设备的电池在远离插头的地方持续足够长的时间的处理器是极其重要的。 来自不同制造商的多种技术和处理器系列诞生了。 在服务器和大型机方面，Intel 值得一提的是其 Core（R 产品系列，即使只有一个物理芯片，它也可以通过模拟多个处理器的存在来欺骗操作系统。

在 Core（R）系列中，处理器进行了重大的内部更改，并采用了称为核心（core）的组件，这些组件具有自己的 **ALU** 和缓存 **L2** 和 **L3**，以及执行指令的其他元素。 这些核心，也称为**逻辑处理器**(logical processors)，允许我们同时并行执行同一程序的不同部分，甚至不同的程序。 *age core* 通过优于其前身的功率处理实现了更低的能源使用。 由于内核并行工作，模拟独立的处理器，我们可以拥有一个多核芯片和一个较低的时钟，从而根据任务获得比具有更高时钟的单核芯片更好的性能。

当然，如此多的改进已经改变了我们进行软件设计的方式。 今天，我们必须考虑并行性来设计合理利用资源而不浪费资源的系统，从而为用户提供更好的体验并节省个人计算机和处理中心的资源。 **并行编程**比以往任何时候都更多地出现在开发人员的日常生活中，而且显然，它永远不会倒退。

本章节包含以下几个主题：

* 为什么使用并行编程？
* 介绍并行化的常见形式
* 在并行编程中通信
* 识别并行编程的问题
* 发现Python的并行编程工具
* 小心Python的**全局解释器锁**（Global Interpreter Lock - GIL）