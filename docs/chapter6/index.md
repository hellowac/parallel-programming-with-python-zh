# 使用并行 Python

在上一章中，我们学习了如何使用 `multiprocessing` 和 `ProcessPoolExecutor` 模块来解决两个案例问题。 本章将介绍命名管道以及如何使用Parallel Python（PP）通过进程执行并行任务。

本章会覆盖下面几个知识点：

* 理解进程间通信
* 了解Parallel Python(PP)
* 在SMP架构上使用PP计算斐波那契序列
* 使用PP创建分布式的网络爬虫