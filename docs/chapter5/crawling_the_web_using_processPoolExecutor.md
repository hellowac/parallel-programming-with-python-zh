# 使用ProcessPoolExecutor模块设计网络爬虫

正如`concurrent.futures`模块提供了`ThreadPoolExecutor`，方便创建和操作多个线程，进程属于`ProcessPoolExecutor`类。 `ProcessPoolExecutor` 类也包含在 `concurrent.futures` 包中，用于实现我们的并行 `Web` 爬虫。 为了实施这个案例研究，我们创建了一个名为 `process_pool_executor_web_crawler.py` 的 `Python` 模块。

代码以前面示例中已知的导入启动，例如`requests`、`Manager` 模块等。 关于任务的定义，以及线程的使用，与上一章的示例相比几乎没有变化，只是现在我们发送数据以通过函数参数形式进行； 参考以下说明：

`group_urls_task`函数定义如下：

```python
def group_urls_task(urls, result_dict, html_link_regex)
```

`crawl_task`函数定义如下：

```python
def crawl_task(url, html_link_regex)
```

现在让我们看一下代码块，其中有一些细微但相关的变化。进入主代码块，我们声明了一个`Manager`类型的对象，现在它将允许**共享队列**，而不仅仅是包含处理结果的字典。为了定义这个包含需要抓取的URL的名为`urls`的队列，我们将使用`Manager.Queue`对象。对于结果字典，我们将使用`Manager.dict`对象，旨在使用一个由代理管理的字典。下面示例代码说明了这些定义。

```python
if __name__ == '__main__':
    manager = Manager()
    urls = manager.Queue()
    urls.put("http://br.bing.com/")
    urls.put("https://github.com")
    result_dict = manager.dict()
```

然后，我们定义了爬虫阶段要使用的正则表达式，并获得了运行该程序的机器的处理器数量，如以下代码所示：

```python
html_link_regex = re.compile('<a\s(?:.*?\s)*?href=[\'"](.*?)[\'"].*?>')
number_of_cpus = cpu_count()
```

In the final chunk, we can notice the consistency in the APIs that are in the concurrent.futures module. The following chunk is exactly the one used in our example using ThreadPoolExecutor, as mentioned in the previous chapter. However, it is enough to change the class to ProcessPoolExecutor by altering the internal behavior and tackling the GIL issue for CPU-bound processes without breaking the code. Check the following chunks; both create ProcessPoolExecutor with workers with limits equal to the number of processors in the machine. The first executor is for grouping the URLs in the dictionary with the standard None value. The second executor proceeds with the crawling stage.

在最后一个代码块中，我们可以注意到`concurrent.futures`模块中的API的一致性。下面这块内容正是我们在上一章中提到的使用`ThreadPoolExecutor`的例子中使用的。然而，通过改变内部行为和解决CPU绑定进程的GIL问题，将该类改为`ProcessPoolExecutor`就足够了，而不会破坏代码。检查以下几个代码块；这两块都创建了`ProcessPoolExecutor`，其工作者的限制等于机器中处理器的数量。第一个executor用于将字典中的`URL`以标准的`None`值分组。第二个executor进行抓取阶段。

下面是第一个executor的一代码块。

```python
with concurrent.futures.ProcessPoolExecutor(max_workers=number_of_cpus) as group_link_processes:
        for i in range(urls.qsize()):
            group_link_processes.submit(group_urls_task, urls, result_dict, html_link_regex)
```

第二个executor的代码块如下：

```python
with concurrent.futures.ProcessPoolExecutor(max_workers=number_of_cpus) as crawler_link_processes:
        future_tasks = {crawler_link_processes.submit(crawl_task, url, html_link_regex): url for url in result_dict.keys()}
        for future in concurrent.futures.as_completed(future_tasks):
            result_dict[future.result()[0]] = future.result()[1]
```

!!! info ""

    使用 `concurrent.futures` 从多线程模式切换到多进程稍微简单一些。

程序运行结果如下图：

```shell
$ python process_pool_executor_web_crawler.py
[SpawnProcess-4] putting url [http://www.google.com] in dictionary...
[SpawnProcess-4] putting url [http://br.bing.com/] in dictionary...
[SpawnProcess-4] putting url [https://duckduckgo.com/] in dictionary...
[SpawnProcess-4] putting url [http://br.search.yahoo.com/] in dictionary...
[SpawnProcess-2] putting url [https://github.com/] in dictionary...
[SpawnProcess-11] crawling url [https://duckduckgo.com/] ...
[SpawnProcess-10] crawling url [http://www.google.com] ...
[SpawnProcess-8] crawling url [https://github.com/] ...
[SpawnProcess-7] crawling url [http://br.bing.com/] ...
[SpawnProcess-9] crawling url [http://br.search.yahoo.com/] ...
[http://www.google.com] with links: [http://www.google.com.hk/imghp?hl=zh-TW&tab=wi...
[http://br.bing.com/] with links: [javascript:void(0);...
[https://duckduckgo.com/] with links: [/about...
[http://br.search.yahoo.com/] with links: [https://br.yahoo.com/...
[https://github.com/] with links: [#start-of-content...
```

## 完整示例

译者注:

```python
import sys
import re
import queue 
from concurrent.futures import ProcessPoolExecutor, as_completed
from multiprocessing import Manager, cpu_count, current_process

import requests


result_dict = {}

def group_urls_task(urls, result_dict, html_link_regex):
    try:
        url = urls.get(True, 0.05)  # true表示阻塞其他线程访问这个队列，0.05表示阻塞的超时时间
        result_dict[url] = None
        print("[%s] putting url [%s] in dictionary..." % (current_process().name, url))
    except queue.Empty:
        print('Nothing to be done, queue is empty')

def crawl_task(url, html_link_regex):
    links = []
    try:
        request_data = requests.get(url)
        print("[%s] crawling url [%s] ..." % (current_process().name, url))
        links = html_link_regex.findall(request_data.text)
    except:
        print(f"error: {sys.exc_info()[0]}")
        raise
    finally:
        return (url, links)

if __name__ == "__main__":

    manager = Manager()
    urls = manager.Queue()
    urls.put('http://www.google.com')
    urls.put('http://br.bing.com/')
    urls.put('https://duckduckgo.com/')
    urls.put('https://github.com/')
    urls.put('http://br.search.yahoo.com/')
    result_dict = manager.dict()

    html_link_regex = re.compile('<a\s(?:.*?\s)*?href=[\'"](.*?)[\'"].*?>')
    number_of_cpus = cpu_count()

    with ProcessPoolExecutor(max_workers=number_of_cpus) as group_link_processes:
        for i in range(urls.qsize()):
            group_link_processes.submit(group_urls_task, urls, result_dict, html_link_regex)
    
    with ProcessPoolExecutor(max_workers=number_of_cpus) as crawler_link_processes:
        future_tasks = {crawler_link_processes.submit(crawl_task, url, html_link_regex): url for url in result_dict.keys()}
        for future in as_completed(future_tasks):
            result_dict[future.result()[0]] = future.result()[1]
    
    for url, links in result_dict.items():
        print(f"[{url}] with links: [{links[0]}...")
```
