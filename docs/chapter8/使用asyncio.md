# 使用asyncio

我们可以将 `asyncio` 定义为一个模块，用于重启 Python 中的异步编程。 `asyncio` 模块允许使用以下元素的组合来实现异步编程：

- **Event loop**: 这已在上一节中定义。 `asyncio` 模块允许每个进程有一个事件循环。
- **Coroutines(协程)**: 正如`asyncio`官方文档中所说，“**协程是一种遵循一定约定的生成器**”。 它最有趣的特性是它可以**在执行期间挂起**以等待外部处理（I/O 中的某些例程）完成, 并在外部处理完成后又可以从原来的位置恢复执行。
- **Futures**: `asyncio` 模块定义了自己的对象 `Future`。 `Futures` 代表一个尚未完成的处理过程。
- **Tasks**: 这是 `asyncio.Future` 的子类，用于**封装**和**管理**协程。

除了这些机制之外，`asyncio` 还为应用程序的开发提供了一系列其他功能，例如传输和协议，它们允许使用 `TCP`、`SSL`、`UDP` 和`管道`等通过通道进行通信。 有关 `asyncio` 的更多信息，请访问 <https://docs.python.org/3.4/library/asyncio.html>{target="_blank"}。

## 理解coroutines和futures

为了能够在`asyncio`中定义一个`coroutine`，我们使用`@asyncio.coroutine`装饰器，并且我们必须利用`yield from`语法来暂停`coroutine`，以便执行一个I/O操作或者其他可能阻塞**事件循环**的计算。但是这种暂停和恢复的机制是如何工作的呢？`Coroutine`与`asyncio.Future`对象一起工作。我们可以把这个操作总结如下:

- 初始化协程，并在内部实例化一个 `asyncio.Future` 对象或将其作为参数传递给协程。
- 在到达使用 `yield from` 的协程点时，协程将暂停以等待在 `yield from` 中引发的计算。 `yield from instance` 等待 `yield from` <coroutine 或 asyncio.Future 或 asyncio.Task\> 的构造。
- 当 `yield from` 中引发的计算结束后，协程执行协程相关的 `asyncio.Future` 对象的 `set_result(<result>)` 方法，告诉事件循环可以恢复协程。

!!! info ""

    当我们使用 `asyncio.Task` 对象封装协程时，我们不需要显式使用 `asyncio.Future` 对象，因为 `asyncio.Task` 对象已经是 `asyncio.Future` 的子类。

## 使用coroutine和asyncio.Future

下面是使用`coroutine`和`asyncio.Future`对象的一些例子：

```python
import asyncio

@asyncio.coroutine
def sleep_coroutine(f):
    yield from asyncio.sleep(2)
    f.set_result("Done!")
```

在上述代码中，我们定义了名为 `sleep_coroutine` 的协程，它接收一个 `asyncio.Future`对象 作为参数。 在`sleep_coroutine`中，我们的协程将运行 `asyncio.sleep(2)` 导致暂停执行并休眠 2 秒； 我们必须观察到 `asyncio.sleep` 函数已经与 `asyncio` 兼容。 因此，它作为未来返回； 然而，由于教学原因，我们包含了作为参数传递的 `asyncio.Future` 对象，以说明如何通过 `asyncio.Future.set_result(<result>)` 在协程中显式完成恢复。

最终，我们有了我们的主函数，我们在其中创建了我们的 `asyncio.Future` 对象，并在 `loop = asyncio.get_event_loop()` 行中，我们从 `asyncio` 创建了一个事件循环实例来执行我们的协程，如下代码所示：

```python
if __name__ == '__main__':
    future = asyncio.Future()
    loop = asyncio.get_event_loop()
    loop.run_until_complete(sleep_coroutine(future))
```

!!! info ""

    任务和协程仅在**事件循环**(event loop)执行时执行。

在最后一行，`loop.run_until_complete(sleep_coroutine(future))`，我们要求我们的事件循环一直运行直到我们的协程完成它的执行。 这是通过 `BaseEventLoop` 类中提供的 `BaseEventLoop.run_until_complete` 方法完成的。

!!! info ""

    在 `asyncio` 中恢复协程的魔法在于 `asyncio.Future` 对象的 `set_result` 方法。 所有要恢复的协程都需要等待`asyncio.Future`执行`set_result`方法。 所以，`asyncio` 的事件循环会知道计算已经结束，它可以恢复协程。

## 使用asyncio.Task

如前所述，`asyncio.Task` 类是 `asyncio.Future` 的子类，旨在管理协程。 让我们检查一个名为 `asyncio_task_sample.py` 的示例代码，其中将创建多个 `asyncio.Task` 对象并在 `asyncio` 的事件循环中分派以执行：

```python
import asyncio

@asyncio.coroutine
def sleep_coro(name, seconds=1):
    print("[%s] coroutine will sleep for %d second(s)…" % (name, seconds))
    yield from asyncio.sleep(seconds)
    print("[%s] done!" % name)
```

我们的协程称为 `sleep_coro`，将接收两个参数：`name`，它将用作我们协程的标识符，以及标准值为 1 的`seconds`，它将指示协程将暂停多少秒。

在主函数中，我们定义了一个列表，其中包含三个类型为 `asyncio.Task` 的对象，名为 `Task-A`，它将休眠 `10` 秒，以及 `Task-B` 和 `Task-C`，它们将分别休眠 `1` 秒。 请参见以下代码：

```python
if __name__ == '__main__':
    tasks = [asyncio.Task(sleep_coro('Task-A', 10)),
             asyncio.Task(sleep_coro('Task-B')),
             asyncio.Task(sleep_coro('Task-C'))
            ]
    loop = asyncio.get_event_loop()
    loop.run_until_complete(asyncio.gather(*tasks))
```

!!! info "由于学习时，python 3已更新到3.10.2 固如下写法也可以"

    ```python
    # python3.10.2 版如下, 使用async、await 关键字
    import asyncio

    async def sleep_coro(name, seconds=1):
        print("[%s] coroutine will sleep for %d second(s)…" % (name, seconds))
        await asyncio.sleep(seconds)
        print("[%s] done!" % name)


    if __name__ == "__main__":
        tasks = [
            asyncio.Task(sleep_coro("Task-A", 10)),
            asyncio.Task(sleep_coro("Task-B")),
            asyncio.Task(sleep_coro("Task-C")),
        ]
        loop = asyncio.get_event_loop()
        loop.run_until_complete(asyncio.gather(*tasks))
    ```

仍然在主函数中，我们使用 `BaseEventLoop` 定义事件循环。 `run_until_complete` 函数； 然而，这个函数接收的参数不超过一个协程，而是一个对 `asyncio.gather` 的调用，它是作为未来返回的函数，附加接收到的协程或未来列表的结果作为参数。 `asyncio_task_sample.py` 程序的输出如以下屏幕截图所示：

![1](../imgs/7-15.png)

值得注意的是，程序的输出按照声明的顺序显示正在执行的任务； 但是，它们都不能阻止事件循环。 这是因为 `Task-B` 和 `Task-C` 睡眠较少并且在 `Task-A` 睡眠 `10` 倍并首先被调度之前结束。 `Task-A` 阻塞事件循环的场景是灾难性的。

## 使用与asyncio不兼容的库

`asyncio` 模块在 `Python` 社区中仍然是最新的。 一些库仍然不完全兼容。 让我们重构上一节示例 `asyncio_task_sample.py` 并将函数从 `asyncio.sleep` 更改为 `time.sleep`。 在不作为未来返回的时间模块中休眠并检查其行为。 我们将 `yield from asyncio.sleep(seconds)` 行更改为 `yield from time.sleep(seconds)`。我们显然需要导入时间模块来使用新的睡眠。 运行该示例，请注意以下屏幕截图中显示的输出中的新行为：

![1](../imgs/7-16.png)

我们可以注意到协程正常初始化，但是由于 `yield from` 语法等待协程或 `asyncio.Future` 而发生错误，并且 `time.sleep` 在其结束时没有生成任何东西。 那么，在这些情况下我们应该如何处理呢？ 答案很简单: 我们需要一个 `asyncio.Future` 对象，然后重构我们的示例。

首先，让我们创建一个函数，该函数将创建一个 `asyncio.Future` 对象以将其返回到 `sleep_coro` 协程中的 `yield from present`。 `sleep_func`函数如下：

```python
def sleep_func(seconds):
    f = asyncio.Future()    
    time.sleep(seconds)    
    f.set_result("Future done!")    
    return f
```

请注意，`sleep_func` 函数在结束时会执行 `f.set_result("Future done!")` 在 `future cause` 中放置一个虚拟结果，因为此计算不会生成具体结果； 它只是一个睡眠功能。 然后，返回一个 `asyncio.Future` 对象，`yield from` 期望它恢复 `sleep_coro` 协程。 以下屏幕截图说明了修改后的 `asyncio_task_sample.py` 程序的输出：

![1](../imgs/7-17.png)

现在所有已分派的任务都执行无误。 可是等等！ 上一个屏幕截图中显示的输出仍然有问题。 请注意，执行顺序内部有些奇怪，因为 **Task-A** 休眠了 10 秒，并在随后两个仅休眠 1 秒的任务开始之前结束。 也就是说，我们的事件循环被任务阻塞了。 这是使用不与 asyncio 异步工作的库或模块的结果。

解决此问题的一种方法是将阻塞任务委托给 `ThreadPoolExecutor`（请记住，如果处理受 **I/O** 限制，则此方法效果很好；如果受 **CPU** 限制，请使用 `ProcessPoolExecutor`。为了我们的舒适，`asyncio` 以一种非常简单的方式支持此机制. 让我们再次重构我们的 `asyncio_task_sample.py` 代码，以便在不阻塞**事件循环**的情况下执行任务。

首先，我们必须删除 `sleep_func` 函数，因为它不再是必需的。 对 `time.sleep` 的调用将由 `BaseEventLoop.run_in_executor` 方法完成。然后让我们按照以下方式重构我们的 `sleep_coro` 协程：

```python
@asyncio.coroutine
def sleep_coro(name, loop, seconds=1):    
    future = loop.run_in_executor(None, time.sleep, seconds)

    print("[%s] coroutine will sleep for %d second(s)..." %        (name, seconds))    
    yield from future    
    print("[%s] done!" % name)
```

!!! info "由于学习时，python 3已更新到3.10.2 固如下写法也可以"

    ```python
    # python3.10.2 版如下, 使用async、await 关键字
    import asyncio

    async def sleep_coro(name, loop, seconds=1):
        future = loop.run_in_executor(None, time.sleep, seconds)
        print("[%s] coroutine will sleep for %d second(s)…" % (name, seconds))
        await future
        print("[%s] done!" % name)


    if __name__ == "__main__":
        loop = asyncio.get_event_loop()
        tasks = [
            asyncio.Task(sleep_coro2("Task-A", loop, 10)),
            asyncio.Task(sleep_coro2("Task-B", loop)),
            asyncio.Task(sleep_coro2("Task-C", loop)),
        ]

        loop.run_until_complete(asyncio.gather(*tasks))
    ```

值得注意的是，协程接收到一个新参数，该参数将是我们在主函数中创建的事件循环，以便使用 `ThreadPoolExecutor` 来响应相同的执行结果。

在这之后，我们有下一行:

```python
future = loop.run_in_executor(None, time.sleep, seconds)
```

在上一行中，调用了 `BaseEventLoop.run_in_executor` 函数，它的第一个参数是一个[执行器](https://docs.python.org/3.4/library/concurrent.futures.html#concurrent.futures.Executor)。 如果它通过 `None`，它将使用 `ThreadPoolExecutor` 作为默认值。 第二个参数是一个回调函数，在本例中是 `time.sleep` 函数，代表我们要完成的计算，最后我们可以传递回调参数。

请注意，`BaseEventLoop.run_in_executor` 方法返回一个 `asyncio.Future` 对象。 然而，通过返回的 `future` 调用 `yield from` 就足够了，并且我们的协程已经准备好了。

记住，我们需要改变程序的主函数，将**事件循环传**(event loop)递给`sleep_coro`。

```python
if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    tasks = [asyncio.Task(sleep_coro('Task-A', loop, 10)),
             asyncio.Task(sleep_coro('Task-B', loop)),
             asyncio.Task(sleep_coro('Task-C', loop))]
    loop.run_until_complete(asyncio.gather(*tasks))    loop.close()
```

让我们看看下面截图中显示的重构后的代码执行情况。

![1](../imgs/7-18.png)

我们得到了它！结果是一致的，事件循环没有被`time.sleep`函数的执行阻塞。
