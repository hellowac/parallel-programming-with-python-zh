# 使用concurrent.futures模块爬取web信息

下一节将通过实现并行 Web 爬虫来使用我们的代码。 在此方案中，我们将使用一个非常有趣的 **Python** 资源，即 `concurrent.futures` 模块中的 `ThreadPoolExecutor`。 在前面的示例中，我们分析了 `parallel_fibonacci.py`，使用了非常原始的线程形式。 此外，在特定时刻，我们不得不手动创建和初始化多个线程。 在较大的程序中，很难管理这种情况。 在这种情况下，有一些机制允许建立一个线程池。 线程池只不过是一个数据结构，它保留了多个先前创建的线程，供某个进程使用。 它旨在**重用线程**，从而避免不必要的线程创建——这是昂贵的。

基本上，如前一章所述，我们将有一个算法将分阶段执行一些任务，这些任务相互依赖。 在这里，我们将研究并行网络爬虫的代码。

导入一些模块并设置日志文件后，我们使用名为 `re` 的内置模块创建了一个**正则表达式**（有关此模块的完整文档可在 <http://docs.python.org/3/howto/regex.html>{target="_blank"}）。 我们将使用它来过滤从抓取阶段返回的页面中的链接。 代码如下：

```python
html_link_regex = re.compile('<a\s(?:.*?\s)*?href=[\'"](.*?)[\'"].*?>')
```

接下来我们创建一个同步队列来模拟输入数据. 然后我们创建一个名为`result_dict`的字典实例. 在此，我们会将 URL 及其各自的链接关联为列表结构。 相关代码如下:

```python
urls = queue.Queue()
urls.put('http://www.google.com')
urls.put('http://br.bing.com/')
urls.put('https://duckduckgo.com/')
urls.put('https://github.com/')
urls.put('http://br.search.yahoo.com/')
result_dict = {}
```

再接下来我们定义一个名为`group_urls_task`的函数,该函数用于从同步队列中抽取出URL并存入`result_dict`的key值中. 另一个应该留意的细节是,我们调用`Queue`的`get`方法是,带了两个参数,第一个参数为`True`表示阻塞其他线程访问这个同步队列,第二个参数是`0.05`表示阻塞的超时时间,这样就防止出现由于同步队列中没有元素而等待太长时间的情况出现. 毕竟,在某些情况下,你不会想化太多的时间来等待新元素的到来. 相关代码如下:

```python
def group_urls_task(urls):
    try:
        url = urls.get(True, 0.05)  # true表示阻塞其他线程访问这个队列，0.05表示阻塞的超时时间
        result_dict[url] = None
        logger.info("[%s] putting url [%s] in dictionary..." % (threading.current_thread().name, url))
    except queue.Empty:
        logging.error('Nothing to be done, queue is empty')
```

现在，我们有了负责完成每个作为参数发送给 `crawl_task` 函数的 `URL` 的抓取阶段的任务。 基本上，抓取阶段是通过获取接收到的 `URL` 指向的页面内的所有链接来完成的。 爬取返回的元组包含第一个元素作为 `crawl_task` 函数接收的 `URL`。 第二步，提取链接列表。 `requests`模块（关于`request`模块的官方文档可以在<https://pypi.python.org/pypi/requests>{target="_blank"}找到）用于从URL获取网页。 代码如下：

```python
def crawl_task(url):
    links = []
    try:
        request_data = requests.get(url)
        logger.info("[%s] crawling url [%s] ..." % (
            threading.current_thread().name, url))
        links = html_link_regex.findall(request_data.text)
    except:
        logger.error(sys.exc_info()[0])
        raise
    finally:
        return (url, links)
```

进一步分析代码，我们将看到创建了一个 `ThreadPoolExecutor` 对象（有关 `ThreadPoolExecutor` 对象的更多信息，请访问 <http://docs.python.org/3.3/library/concurrent.futures.html#concurrent.futures.ThreadPoolExecutor>{target="_blank"} ) 在 `concurrent.futures` 模块中有特色。 在这个 `ThreadPoolExecutor` 对象的构造函数中，我们可以定义一个名为 `max_workers` 的参数, 该参数决定了线程池中的线程数。 在从同步队列中删除 `URL` 并将键插入到 `result_dict` 的阶段，选择是使用三个工作线程。 数量将根据问题而有所不同。 在定义 `ThreadPoolExecutor` 并使用 `with` 语句来保证结束线程之后，这些线程将在 `with` 语句范围的输出中执行。 在 `ThreadPoolExecutor` 对象的范围内，我们在同步队列中对其进行迭代，并通过 `submit` 方法分派它为包含 `URL` 的队列线程执行引用。 总而言之，`submit` 方法为线程的执行安排了一个可调用对象，并返回一个包含为执行创建的安排的 `Future` 对象。 `submit` 方法接收一个可调用对象及其参数； 在我们的例子中，可调用项是任务 `group_urls_task`，参数是对同步队列函数的引用。 调用这些参数后，池中定义的工作线程将以并行、异步的方式执行预订。 代码如下：

```python
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as group_link_threads:
    for i in range(urls.qsize()):
        group_link_threads.submit(group_urls_task, urls)
```

在前面的代码之后，我们又创建了一个`ThreadPoolExecutor`； 但是这一次，我们要使用上一阶段`group_urls_task`生成的`key`来运行爬虫阶段。 这一次我们所使用的代码有些不同:

```python
future_tasks = {
    crawler_link_threads.submit(crawl_task, url): url  
    for url in result_dict.keys()}
```

我们映射了一个名为`future_tasks`的临时字典。它将包含`submit`的结果，通过`result_dict`中的每个`URL`来完成。也就是说，对于每个`key`，我们在`future_tasks`中创建一个条目。在映射之后，我们需要收集`submit`的结果，因为它们是用一个循环执行的，它使用`concurrent.futures.as_completed(fs，timeout=None)`方法在`future_tasks`中寻找已完成的条目。这个调用返回一个`Future`对象实例的迭代器。因此，我们可以在已经派发的预订所处理的每个结果中进行迭代。在`ThreadPoolExecutor`的最后，对于爬行线程，我们使用`Future`对象的`result()`方法。在抓取阶段的情况下，它返回结果元组。通过这种方式，我们在`future_tasks`中生成最后的条目，如下面的截图所示。

```shell
$ python temp2.py
2023-03-01 15:53:51,289 - [ThreadPoolExecutor-0_0] putting url [http://www.google.com] in dictionary...
2023-03-01 15:53:51,289 - [ThreadPoolExecutor-0_1] putting url [http://br.bing.com/] in dictionary...
2023-03-01 15:53:51,290 - [ThreadPoolExecutor-0_0] putting url [https://duckduckgo.com/] in dictionary...
2023-03-01 15:53:51,290 - [ThreadPoolExecutor-0_2] putting url [https://github.com/] in dictionary...
2023-03-01 15:53:51,290 - [ThreadPoolExecutor-0_1] putting url [http://br.search.yahoo.com/] in dictionary...
2023-03-01 15:53:51,334 - Starting new HTTP connection (1): 127.0.0.1:7890
2023-03-01 15:53:51,408 - Starting new HTTPS connection (1): duckduckgo.com:443
2023-03-01 15:53:51,411 - Starting new HTTP connection (1): 127.0.0.1:7890
2023-03-01 15:53:51,584 - http://127.0.0.1:7890 "GET http://www.google.com/ HTTP/1.1" 200 6588
2023-03-01 15:53:51,585 - [ThreadPoolExecutor-1_0] crawling url [http://www.google.com] ...
2023-03-01 15:53:51,621 - Starting new HTTPS connection (1): github.com:443
2023-03-01 15:53:51,625 - http://127.0.0.1:7890 "GET http://br.bing.com/ HTTP/1.1" 302 0
2023-03-01 15:53:51,628 - Resetting dropped connection: 127.0.0.1
2023-03-01 15:53:51,704 - https://duckduckgo.com:443 "GET / HTTP/1.1" 200 None
2023-03-01 15:53:51,706 - [ThreadPoolExecutor-1_2] crawling url [https://duckduckgo.com/] ...
2023-03-01 15:53:51,822 - Starting new HTTP connection (1): 127.0.0.1:7890
2023-03-01 15:53:51,894 - http://127.0.0.1:7890 "GET http://www.bing.com/?cc=br HTTP/1.1" 200 None
2023-03-01 15:53:51,962 - https://github.com:443 "GET / HTTP/1.1" 200 None
2023-03-01 15:53:51,978 - [ThreadPoolExecutor-1_1] crawling url [http://br.bing.com/] ...
2023-03-01 15:53:52,045 - [ThreadPoolExecutor-1_0] crawling url [https://github.com/] ...
2023-03-01 15:53:52,223 - http://127.0.0.1:7890 "GET http://br.search.yahoo.com/ HTTP/1.1" 301 25
2023-03-01 15:53:52,225 - Starting new HTTPS connection (1): br.search.yahoo.com:443
2023-03-01 15:53:52,697 - https://br.search.yahoo.com:443 "GET / HTTP/1.1" 200 17530
2023-03-01 15:53:52,859 - [ThreadPoolExecutor-1_2] crawling url [http://br.search.yahoo.com/] ...
```

又一次,我们可以发现每个线程池中的线程执行是乱序的,但这不重要,重要的是,`resul\_dict`中输出的内容就是最终结果.

## 完整代码

译者注:

```python
import sys
import re
import logging, threading
import queue 
from concurrent.futures import ThreadPoolExecutor

import requests

logger = logging.getLogger()
logger.setLevel(logging.DEBUG)
formatter = logging.Formatter('%(asctime)s - %(message)s')

ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)
ch.setFormatter(formatter)
logger.addHandler(ch)

html_link_regex = re.compile('<a\s(?:.*?\s)*?href=[\'"](.*?)[\'"].*?>')

urls = queue.Queue()
urls.put('http://www.google.com')
urls.put('http://br.bing.com/')
urls.put('https://duckduckgo.com/')
urls.put('https://github.com/')
urls.put('http://br.search.yahoo.com/')

result_dict = {}

def group_urls_task(urls):
    try:
        url = urls.get(True, 0.05)  # true表示阻塞其他线程访问这个队列，0.05表示阻塞的超时时间
        result_dict[url] = None
        logger.info("[%s] putting url [%s] in dictionary..." % (threading.current_thread().name, url))
    except queue.Empty:
        logging.error('Nothing to be done, queue is empty')

def crawl_task(url):
    links = []
    try:
        request_data = requests.get(url)
        logger.info("[%s] crawling url [%s] ..." % (threading.current_thread().name, url))
        links = html_link_regex.findall(request_data.text)
    except:
        logger.error(sys.exc_info()[0])
        raise
    finally:
        return (url, links)
    
with ThreadPoolExecutor(max_workers=3) as group_link_threads:
    for i in range(urls.qsize()):
        group_link_threads.submit(group_urls_task, urls)

with ThreadPoolExecutor(max_workers=3) as crawler_link_threads:
    future_tasks = {
        crawler_link_threads.submit(crawl_task, url): url  
        for url in result_dict.keys()}
```
