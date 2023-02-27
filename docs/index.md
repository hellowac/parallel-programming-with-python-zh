# parallel programming with python

使用 python 进行并行编程

使用强大的 python 环境开发高效的并行系统

-- Jan Palach

## 关于作者

Jan Palach 从事软件开发已有 13 年，曾与科学界合作使用 C++、Java 和 Python 技术的私人公司的可视化和后端。 Jan 拥有巴西里约热内卢 Estácio de Sá 大学的信息系统学位，以及巴拉那州立联邦理工大学的软件开发研究生学位。
目前，他在一家实施 C++ 系统的电信行业的私营公司担任高级系统分析师； 但是，他喜欢尝试使用 Python 和 Erlang 来获得乐趣——这是他的两大技术爱好。
他天生好奇，喜欢挑战和学习新技术、结识新朋友以及了解不同的文化。

## 致谢

我不知道在如此紧迫的期限内写一本书有多难，我生活中发生了很多其他事情。 我不得不把写作融入我的日常生活中，
照顾我的家人、空手道课程、工作、暗黑破坏神 III 等等。 这项任务并不容易； 然而，考虑到根据我的经验，我已经专注于最重要的事情，我希望我已经产生了高质量的内容来取悦大多数读者。

我要致谢的人名单很长，我只需要一本书就可以了。 所以，我要感谢一些我经常联系的人，他们以直接或间接的方式在整个探索过程中帮助了我。

我的妻子 Anicieli Valeska de Miranda Pertile，我选择与她分享我的爱并收集牙刷直到生命的尽头，是她让我有时间创作这本书，并没有让我在我想我的时候放弃 做不到。 在我作为一个人的成长过程中，我的家人一直对我很重要，并教会了我善良的道路。

我要感谢 Fanthiane Ketrin Wentz，她不仅是我最好的朋友，还指导我学习武术，教会我一生都将秉持的价值观——她是我的榜样。 Lis Marie Martini，亲爱的朋友，为本书提供了封面，她是一位了不起的摄影师和动物爱好者。

非常感谢我以前的英语老师、审校者和校对者玛丽娜·梅洛 (Marina Melo)，她在本书的写作过程中提供了帮助。 感谢审稿人和我的好友 Vitor Mazzi 和 Bruno Torres，他们对我的职业发展做出了巨大贡献，现在仍然如此。

特别感谢 Rodrigo Cacilhas、Bruno Bemfica、Rodrigo Delduca、Luiz Shigunov、Bruno Almeida Santos、Paulo Tesch (corujito)、Luciano Palma、Felipe Cruz 以及我经常与之谈论技术的其他人。 特别感谢 Turma B.

非常感谢 Guido Van Rossum 创建了 Python，它将编程变成了令人愉快的事情； 我们需要更多这样的东西，而不是设置/获取。

## 关于审稿人

**Cyrus Dasadia** 在 AOL 和 InMobi 等组织担任 Linux 系统管理员已有十多年。 他目前正在开发 CitoEngine，这是一种完全用 Python 编写的开源警报管理服务。

**Wei Di** 是 eBay 研究实验室的研究科学家，专注于大型电子商务应用的高级计算机视觉、数据挖掘和信息检索技术。 她的兴趣包括大规模数据挖掘、商品销售中的机器学习、电子商务的数据质量、搜索相关性以及排名和推荐系统。 她在模式识别和图像处理方面也有多年的研究经验。 她于 2011 年在普渡大学获得博士学位，重点研究数据挖掘和图像分类。

**Michael Galloy** 在 Tech-X Corporation 担任研究数学家，参与使用 IDL 和 Python 进行科学可视化。 在此之前，他曾为 Research Systems, Inc.（现为 Exelis Visual Information Solutions）教授各级 IDL 编程和咨询工作五年。 他是 Modern IDL (modernidl.idldev.com) 的作者，并且是多个开源项目的创建者/维护者，包括 IDLdoc、mgunit、dist_tools 和 cmdline_tools。 他为他的网站 michaelgalloy.com 撰写了 300 多篇关于 IDL、科学可视化和高性能计算的文章。 他是 NASA 的首席研究员，资助了使用 IDL 进行远程数据探索以实现 IDL 中的 DAP 绑定，以及使用现代图形卡加速曲线拟合的快速模型拟合工具套件。

**Ludovic Gasc** 是欧洲知名的开源 VoIP 和统一通信公司 Eyepea 的高级软件集成工程师。 在过去的五年中，Ludovic 基于 Python（Twisted 和现在的 AsyncIO）和 RabbitMQ 为电信开发了冗余分布式系统。

他还是多个 Python 库的贡献者。 有关这方面的更多信息和详细信息，请参阅 <https://github.com/GMLudo>。

**Kamran Husain** 在计算机行业工作了大约 25 年，为电信和石油行业编程、设计和开发软件。 他喜欢在空闲时间涉足漫画。

**Bruno Torres** 已经工作了十多年，解决了多个领域的各种计算问题，涉及客户端和服务器端应用程序的组合。 Bruno 拥有巴西里约热内卢 Universidade Federal Fluminense 的计算机科学学位。

在数据处理、电信系统以及应用程序开发和媒体流方面工作后，他从 Java 和 C++ 数据处理系统开始开发了许多不同的技能，通过解决电信行业的可扩展性问题和使用 Lua 简化大型应用程序定制，到 为移动设备和支持系统开发应用程序。

目前，他在一家大型媒体公司工作，开发了许多通过 Internet 为桌面浏览器和移动设备传送视频的解决方案。

他热衷于学习不同的技术和语言、结识新朋友，并热爱解决计算问题的挑战。

## 前言

几个月前，也就是 2013 年，Packt Publishing 的专业人士联系我，让我写一本关于使用 Python 语言进行并行编程的书。 我以前从未想过要写一本书，也不知道即将进行的工作； 构思这项工作会有多复杂，以及在我目前的工作中将它融入我的工作时间表会有什么感觉。 虽然我考虑了几天这个想法，但我最终还是接受了这个任务，并对自己说这将是一次大量的个人学习，也是一个向全世界的观众传播我的 Python 知识的绝好机会，因此 ，希望在我这一生的旅程中留下宝贵的遗产。

这项工作的第一部分是概述其主题。 取悦所有人并不容易； 但是，我相信我已经在这本迷你书提出的主题上取得了很好的平衡，我打算在其中介绍结合理论和实践的 Python 并行编程。 我在这项工作中冒了风险。 我使用了一种新的格式来展示如何解决问题，其中示例在第一章中定义，然后使用本书中提供的工具来解决。 我认为这是一种有趣的格式，因为它允许读者分析和质疑 Python 提供的不同模块。

所有章节都结合了一些理论，从而构建了上下文，为您提供一些基本知识来理解文本的实际部分。 我真诚地希望这本书对那些探索 Python 并行编程世界的人有用，因为我一直努力专注于高质量的写作。

## 本书涵盖的内容

第 1 章，上下文化并行、并发和分布式编程，涵盖了并行编程模型的概念、优点、缺点和含义。 此外，本章还公开了一些 Python 库来实现并行解决方案。

第 2 章，设计并行算法，介绍了有关设计并行算法的一些技术的讨论。

第 3 章，识别可并行化问题，介绍了一些问题示例，并分析了这些问题是否可以拆分为并行部分。

第 4 章，使用 threading 和 concurrent.futures 模块，解释了如何使用 threading 和 concurrent.futures 模块实现第 3 章中提出的每个问题，识别可并行化的问题。

第 5 章，使用 Multiprocessing 和 ProcessPoolExecutor，介绍如何使用 multiprocessing 和 ProcessPoolExecutor 实现第 3 章中提出的每个问题，识别可并行化的问题。

第 6 章，使用并行 Python，介绍如何使用并行 Python 模块实现第 3 章中提出的每个问题，识别可并行化的问题。

第 7 章，使用 Celery 分配任务，解释了如何使用 Celery 分布式任务队列实现第 3 章中提出的每个问题，识别可并行化的问题。

第 8 章，异步执行，解释了如何使用 asyncio 模块和有关异步编程的概念。

## 这本书需要你掌握什么

Python 编程的先前知识是必要的，因为 Python 教程不会包含在本书中。 欢迎了解并发和并行编程，因为本书是为刚开始从事此类软件开发的开发人员设计的。 关于软件，有必要获得以下内容：

- 第 8 章“异步处理”需要 Python 3.3 和 Python 3.4（仍在开发中）
- 需要读者选择的任何代码编辑器
- 应安装并行 Python 模块 1.6.4
- 第 5 章使用 Multiprocessing 和 ProcessPoolExecutor 需要 Celery 框架 3.1
- 需要读者选择的任何操作系统

## 这本书是给谁的

本书是关于使用 Python 进行并行编程的紧凑讨论。 它为初级和中级 Python 开发人员提供工具。 本书适合那些愿意全面了解使用 Python 开发并行/并发软件并学习不同 Python 替代方案的人。 到本书结束时，您将使用各章中提供的信息扩大您的工具箱。

## 惯例

在本书中，您会发现许多区分不同类型信息的文本风格。 以下是这些样式的一些示例，以及对它们含义的解释。

文中代码如下：“为了举例说明multiprocessing.Pipe对象的使用，我们将实现一个创建两个进程A和B的Python程序。”

这段代码如下：

```python
def producer_task(conn):    
    value = random.randint(1, 10)    
    conn.send(value)    
    print('Value [%d] sent by PID [%d]' % (value, os.getpid()))    
    conn.close()
```

任何命令行输入或输出都写成如下：

**`$celery –A tasks –Q sqrt_queue,fibo_queue,webcrawler_queue worker --loglevel=info`**

## 英文原文

参考: [Parallel Programming with Python](./files/Parallel%20Programming%20with%20Python.pdf){target="_blank"}
