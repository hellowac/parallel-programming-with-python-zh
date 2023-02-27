# 在并行编程通信

在并行编程中，workers被送来执行任务，而执行任务常常需要建立通信，以便可以合作解决一个问题。
在大多数情况下，通信以一种可以在workers之间进行数据交换的方式被建立。当说到并行编程，有两种通信方式广为人知：共享状态和消息传递。在下面的章节中，将对这两种方式进行简要描述。

## 理解共享状态

在workers中最有名的一种通信方式就是共享状态。分享状态似乎是一个简单的使用，但是这会有许多的陷阱，因为若其中某个进程对共享的资源执行了一项无效的操作会影响到所有其它的进程，从而导致一个不好的结果。这也是使在多台计算机之间进行分布式的程序成为不可能的显而易见的原因。

为了说明这一点，我们将使用一个真实的案例。假设你是一个具体的银行的一个客户，而这个银行只用一个收银员。当你去银行，你必须要排队等到轮到你的时候。当你在队列中时，你注意到收银员一次只能为一个顾客服务，而收银员不可能同时为两个顾客提供服务而不出错。电脑运算拥有多种手段来以可控的方式访问数据，如mutex（互斥？）。

Mutex可以理解为一种特殊的过程变量，表示了访问数据的可靠性等级。也就是说，在真实世界的栗子中，顾客有一个编号，在某一特定的时刻，这个编号将会被激活，然后收银员仅对于这个顾客提供服务。在进程结束时，该名顾客将会释放收银员让其为下一个顾客服务，以此类推。

> 在某些情况下，当程序正在运行时，在一个变量中数据会有一个常数值，数据仅仅以只读的目的被分享。所以访问控制不是必须的，因为永远不会出现完整性问题。

## 理解信息传递

运用消息传递是为了避免来自共享状态带来的数据访问控制以及同步的问题。消息传递包含一种在运行的进程中进行消息交换的机制。每当我们用分布式架构开发程序的时候，就能见到消息传递的使用，在网络中，消息交换被放在一个重要的位置。Erlang等语言，在它的并行体系结构中，就是使用这个模型来实现通信。由于每次数据交换都复制一份数据的拷贝，因此不会出现并发访问的问题。尽管内存使用看起来比共享内存状态要高，但是这个模型还是有一些优势的。优势如下：

* 缺乏数据的一致性访问
* 数据即可在本地交换（不同的进程）也能在分布式环境中交换
* 不太可能出现扩展性问题,并且允许不同系统相互写作。
* 一般来说，据程序员来说易于维护。