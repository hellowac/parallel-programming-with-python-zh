# 为什么使用并行编程

自从计算系统发展以来，它们已经开始提供一个能使我们以并行的方式运行特定程序的独立部分的机制，从而增强响应和总体性能。 此外，我们可以很容易的就验证配备有多个处理器以及多核的机器。那么，为什么不利用这种架构呢？

并行编程在所有系统开发环境中都是可实现的，从智能手机和平板电脑到研究中心的重型计算。 并行编程的坚实基础将使开发人员能够优化应用程序的性能。 这会增强用户体验以及计算资源的消耗，从而减少完成复杂任务的处理时间。

举一个并行性的例子，让我们想象一个场景，在这个场景中，有一些任务，其中一个任务是从数据库中检索一些信息，而这个数据库规模又很大。再假如，这个应用还需要顺序执行，在这个应用中，这些任务必须以一定的逻辑顺序，一个接一个的执行。当用户请求数据时，在返回的数据没有结束之前，其它系统将一直被阻塞。然而，利用并行编程，我们将会创造一个新的worker来在数据库中查询信息，而不会阻塞这个应用的其它功能，从而提高它的使用。

举一个并行性的例子，让我们想象一个场景，在这个场景中，除了其他任务之外，应用程序还从数据库中查询信息，并且这个数据库有相当大的规模。 还要考虑，应用程序中的任务是按逻辑顺序一个接一个地执行的。 当用户请求数据时，系统的其余部分将被阻塞，直到数据返回未结束。 然而，利用并行编程，我们将被允许创建一个新的 **worker**，它将在这个数据库中查找信息而不阻塞应用程序中的其他功能，从而增强它的使用。