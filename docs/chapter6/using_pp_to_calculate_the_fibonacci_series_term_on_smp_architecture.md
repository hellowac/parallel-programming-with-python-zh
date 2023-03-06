# 在SMP架构上使用pp模块计算斐波那契序列

是时候开始行动了！ 让我们解决涉及在 SMP 架构中使用 PP 的多个输入的斐波那契数列的案例研究。 我正在使用配备双核处理器和四个线程的笔记本电脑。

我们将为这个实现只导入两个模块，`os` 和 `pp`。`os` 模块将仅用于获取正在执行的进程的 `PID`。 我们将有一个名为 `input_list` 的列表，其中包含要计算的值和一个用于对结果进行分组的字典，我们将其称为 `result_dict`。 然后，我们转到代码块如下：

```python
import os, pp

input_list = [4, 3, 8, 6, 10]
result_dict = {}
```

然后，我们定义一个名为 `fibo_task` 的函数，它将由**并行进程**执行。 它将是我们通过 `Server` 类的提交方法传递的 `func` 参数。 该函数与前几章相比没有重大变化，除了现在通过使用元组封装参数中接收的值以及包含 `PID` 和计算的 `Fibonacci` 项的消息来完成返回。 看看下面的完整函数：

```python
def fibo_task(value):    
    a, b = 0, 1     
    for item in range(value):         
        a, b = b, a + b     
        message = "the fibonacci calculated by pid %d was %d" % (os.getpid(), a)     
    
    return (value, message)
```

下一步是定义我们的回调函数，我们将其称为 `aggregate_results`。 一旦 `fibo_task` 函数返回其执行结果，就会调用回调函数。 它的实现非常简单，只显示一条状态消息，随后在 `result_dict` 中生成一个输入，其中包含传递给 `fibo_dict` 函数的值作为键，结果是计算斐波那契项的过程返回的消息。 以下代码是`aggregate_results`函数的完整实现：

```python
def aggregate_results(result):    
    print "Computing results with PID [%d]" % os.getpid()

    result_dict[result[0]] = result[1]
```

现在，我们有两个函数要定义。 我们必须创建一个 `Server` 类的实例来分派任务。 以下代码行创建一个服务器实例：

```python
job_server = pp.Server()
```

在前面的示例中，我们使用标准值作为参数。 在下一节中，我们将使用一些可用的参数。

现在我们有了 `Server` 类的一个实例，让我们迭代 `input_list` 的每个值，通过提交调用分派 `fibo_task` 函数，将需要导入的模块作为参数传递给 `args` 元组中的输入值，以便函数 正确执行并且回调注册 `aggregate_results`。 参考以下代码块：

```python
for item in input_list:    
    job_server.submit(fibo_task, (item,), modules=('os',),          
        callback=aggregate_results)
```

最后，我们必须等到所有派发的任务结束。 因此，我们可以使用`Server`类的`wait`方法，如下：

```python
job_server.wait()
```

!!! info ""

    除了使用回调函数之外，还有另一种方法可以获得已执行函数的返回值。 `submit` 方法返回一个对象类型，`pp._Task`，当执行结束时，它包含了执行的结果。

最后，我们将通过字典迭代打印条目的结果，如下所示：

```python
print "Main process PID [%d]" % os.getpid() 
for key, value in result_dict.items():    
    print "For input %d, %s" % (key, value)
```

以下屏幕截图说明了程序的输出：

```shell
$ python feibonacci_pp_smp.py 
Computing results with PID [21058]
Computing results with PID [21058]
Computing results with PID [21058]
Computing results with PID [21058]
Computing results with PID [21058]
Main process PID [21058]
For input 4, the fibonacci calculated by pid 21059 was 3
For input 3, the fibonacci calculated by pid 21060 was 2
For input 6, the fibonacci calculated by pid 21062 was 8
For input 8, the fibonacci calculated by pid 21061 was 21
For input 10, the fibonacci calculated by pid 21065 was 55
```

## 完整示例

译者注: `feibonacci_pp_smp.py`

```python
import os, pp

input_list = [4, 3, 8, 6, 10]
result_dict = {}

def fibo_task(value):    
    a, b = 0, 1     
    for item in range(value):         
        a, b = b, a + b     
        message = "the fibonacci calculated by pid %d was %d" % (os.getpid(), a)     

    return (value, message)

def aggregate_results(result):    
    print("Computing results with PID [%d]" % os.getpid())

    result_dict[result[0]] = result[1]

job_server = pp.Server()

for item in input_list:    
    job_server.submit(fibo_task, (item,), modules=('os',),       # 这里增加新的模块需要添加进来。        
        callback=aggregate_results)

job_server.wait()

print("Main process PID [%d]" % os.getpid())

for key, value in result_dict.items():    
    print("For input %d, %s" % (key, value))
```
