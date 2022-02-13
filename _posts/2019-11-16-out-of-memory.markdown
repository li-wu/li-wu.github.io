---
title: 内存泄露问题排查
date: 2019-11-16
categories: performance
---

最近在做的一个项目中遇到了OOM的问题，排查和反复测试花了很多时间和精力在上面，这里稍微梳理记录一下。 

这个项目是用来生成各种测试数据的，性能测试团队在使用的时候发现当设置每天生成的数据量比较大的时候比如1TB，就会有内存泄露的问题存在。该项目生成数据的流程大概是先启动多个进程，然后进程从一个共享的队列里面拿要生成数据的任务，数据生成之后要么直接发送到目的地（stdout/file/socket）要么发到另外一个队列中。

```python
process = multiprocessing.Process(target=_proc_worker_do_work, args=(worker_queue))
process.start()

def _proc_worker_do_work(worker_queue):
    item = worker_queue.get()
    item.run()
    worker_queue.task_done()
```

上面只是伪代码。这个时候得用一些内存分析工具来看看，比如[memory profiler](https://pypi.org/project/memory-profiler/),实际好像并没有发现什么问题。

第一想法是可能是程序里面的问题，因为原来的用法并没有用进程池去管理进程，而是在自己来控制。也就是可能会有和这样类似的问题：[https://stackoverflow.com/questions/21485319/high-memory-usage-using-python-multiprocessing](https://stackoverflow.com/questions/21485319/high-memory-usage-using-python-multiprocessing)。改成了用process pool实现重新测试了一下，内存的使用挺正常的。发布一个小的版本之后其它用户反应问题还是存在。

既然问题还存在那肯定不是第一个改动能解决的，所以先把之前的改动撤回了。 第二个想到的点是日志，产生少量的数据的时候日志不会带来大的影响，但是当产生的数据很大并且这个数据生成的服务是一个常驻服务的时候，日志的影响还是非常大的，所以在类生产环境上就把日志默认关掉了，这样改动上去多测试了几次内存的使用好像一直都比较问题，20个进程每个进程的内存使用不到100MB，而且没有随着时间的推移增长。这次在测试环境中让用户再次验证，但是和我自己的测试还是有差别，也就是说问题依然存在。

这就非常奇怪了，因为在测试的过程中也有一些新的代码改动进来，包括Python3的迁移等等，所以基本上就去一个个地排查。最后发现是一个非常奇葩的原因导致的。

用户在使用我们的docker image时候需要依赖一个解析`json`的库叫`ujson` ，但是这个库好像很久都没人维护了，而且有一个[bug](https://github.com/esnme/ultrajson/issues/326)，就是在`alpine`的image上没法用pip安装，这个issue里面有人给出了一个解决办法是通过源代码装，所以我们在制作docker镜像的时候也采用这种方式来装的。还有一个关键的问题是用这种方式在本地构建镜像的时候没办法成功，所以我测试的时候都会把这一段给注释掉，这就是为什么我本地构建的镜像总是能测试成功，但是CI构建出来的镜像在用户环境中就会有问题。但是这样安装会给我们的程序带来OOM的问题。所以最终的[解决方案](https://github.com/splunk/eventgen/pull/336/files)也非常简单。因为他们还是使用Python2，所以只在Python2的环境下装这个而在Python3的环境中不装，然后使用`git+https`而非`git+git`来保证本地环境能构成镜像成功，并且和CI上保持一致。

一些教训：

* 保持本地测试环境和线上测试环境一致
* 调试找问题的时候要善用工具，多看日志

-End-
