---
title: Python Thread and Threadpool
date: 2019-04-09
categories: Python
---

### 线程模块

Python虽然有GIL的限制，但是通过多线程处理IO密集的任务还是能够大大提高程序执行效率。如果使用Python2来写多线程的程序，Python中提供了`thread`和`threading`模块，`thread`模块提供了多线程底层支持，以低级原始的方式来处理和控制线程，使用起来相对复杂。而`threading`模块基于`thread`进行封装，将线程的操作对象化，所以更推荐使用`threading`来编写多线程程序。`threading`模块相对`thread`模块的优势：

* `threading`模块对同步原语的支持完善和丰富。
* `threading`模块在主线程和子线程的交互上更友好。`join`方法可以阻塞当前上下文环境的线程，知道调用此方法的线程终止或者到达指定的`timeout`。
* `thread`模块不支持守护进程。
* Pyhon3中已经不存在`thread`模块，`thread`模块被重命名成了`_thread`。

总之使用`threading`来写多线程程序就对了。

### 线程的创建

可以通过两种方式来创建线程：一种是继承`Thread`类，重写`run`方法；另一种是创建一个`threading.Thread`对象，在初始化函数(`__init__()`)中传入可调用对象。

下面是一个通过第二种方式创建线程的示例：

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import time, threading

def loop():
	print 'thread %s is running...' % threading.current_thread().name
	n = 0
	while n < 5:
		n = n + 1
		print 'thread %s >>> %s' % (threading.current_thread().name, n)
		time.sleep(1)
	print 'thread %s ended.' % threading.current_thread().name

print 'thread %s is running...' % threading.current_thread().name
t = threading.Thread(target=loop, name='LoopThread')
t.start()
t.join()
print 'thread %s ended.' % threading.current_thread().name
```

不过更多的关于多线程的教程或者示例都是通过实现一个生产者-消费者模型来展示的，比如：

```python
'''
Standard Producer/Consumer Threading Pattern
'''

import time 
import threading 
import Queue 

class Consumer(threading.Thread): 
    def __init__(self, queue): 
        threading.Thread.__init__(self)
        self._queue = queue 

    def run(self):
        while True: 
            # queue.get() blocks the current thread until 
            # an item is retrieved. 
            msg = self._queue.get() 
            # Checks if the current message is 
            # the "Poison Pill"
            if isinstance(msg, str) and msg == 'quit':
                # if so, exists the loop
                break
            # "Processes" (or in our case, prints) the queue item   
            print "I'm a thread, and I received %s!!" % msg
        # Always be friendly! 
        print 'Bye byes!'


def Producer():
    # Queue is used to share items between
    # the threads.
    queue = Queue.Queue()

    # Create an instance of the worker
    worker = Consumer(queue)
    # start calls the internal run() method to 
    # kick off the thread
    worker.start() 

    # variable to keep track of when we started
    start_time = time.time() 
    # While under 5 seconds.. 
    while time.time() - start_time < 5: 
        # "Produce" a piece of work and stick it in 
        # the queue for the Consumer to process
        queue.put('something at %s' % time.time())
        # Sleep a bit just to avoid an absurd number of messages
        time.sleep(1)

    # This the "poison pill" method of killing a thread. 
    queue.put('quit')
    # wait for the thread to close down
    worker.join()


if __name__ == '__main__':
    Producer()
```

### 线程池

线程的生命周期分为五个状态：创建，就绪，运行，阻塞和终止。在多数多线程处理的情景中，如果线程不能被重用，意味着每次线程创建都要经历启动，运行，销毁三个过程。要提高线程处理的效率，可以采用线程池。

Python2中提供了一个`threadpool`的线程池模块： [https://pypi.org/project/threadpool/](<https://pypi.org/project/threadpool/>)，这个模块已经被废弃，这里讲这个也是为了如果在老的项目中看到这个的时候知道有这么个东西。

```Python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import time
import threadpool

def save_callback(request, result):
    # 第1个参数是request，可以访问request.requestID
    # 第2个参数是request执行完的结果
    print(request.requestID, result)
    with open('result.txt', 'a') as f:
        f.write(result + '\n')


def get_user_info(uid, sex, name, age):
    time.sleep(0.3)
    return "{0},{1},{2},{3}".format(uid, sex, name, age)


if __name__ == '__main__':
    num = 100
    para_list = [[i, 'male'] for i in range(1, num)]
    users = list()
    for i in range(1, num):
        user = {'name'.format(i): 'user{0}'.format(i),
                'age': i}
        users.append(user)
    params = zip(para_list, users)
    # print(params)
    # 形如[([1, 'male'], {'age': 1, 'name': 'user1'}), ...]的参数列表

    pool_size = 10
    # create threadpool with size 10
    pool = threadpool.ThreadPool(pool_size)
    # create workRequest
    requests = threadpool.makeRequests(get_user_info, params, save_callback)
    # put requests to threadpool
    [pool.putRequest(req) for req in requests]
    pool.wait()
    # exit after all tasks are done
    pool.dismissWorkers(pool_size, do_join=True)
```

具体的使用文档可以参考如下链接：

<https://zhuanlan.zhihu.com/p/34403034>

<https://blog.csdn.net/hehe123456zxc/article/details/52258431>

在Python3中推荐使用`concurrent.futures` 模块中的`threadpoolexecutor`： <https://docs.python.org/3/library/concurrent.futures.html#threadpoolexecutor>

### multiprocessing.dummy

在日常的脚本中可能你并不需要复杂的`ThreadPool`, 队列以及维护队列两端的一系列操作。你需要的仅仅只是开多个线程然后让他们并行地去处理任务， 然后获得处理后的结果。这种情况下可以用到`multiprocessing.dummy`:  [https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing.dummy](https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing.dummy)

`dummy` 是 `multiprocessing` 模块的完整克隆，唯一的不同在于 `multiprocessing` 作用于进程，而 `dummy` 模块作用于线程，因此也包括了 Python 所有常见的多线程限制。

比如有一个需求是把给定的一系列网页内容抓取下来，采用`multiprocessing.dummy`可以这么写：

```python

import urllib2 
from multiprocessing.dummy import Pool as ThreadPool 

urls = [
    'http://www.python.org', 
    'http://www.python.org/about/',
    'http://www.onlamp.com/pub/a/python/2003/04/17/metaclasses.html',
    'http://www.python.org/doc/',
    'http://www.python.org/download/',
    'http://www.python.org/getit/',
    'http://www.python.org/community/',
    'https://wiki.python.org/moin/',
    'http://planet.python.org/',
    'https://wiki.python.org/moin/LocalUserGroups',
    'http://www.python.org/psf/',
    'http://docs.python.org/devguide/',
    'http://www.python.org/community/awards/'
    # etc.. 
    ]

# Make the Pool of workers
pool = ThreadPool(4) 
# Open the urls in their own threads and return the results
results = pool.map(urllib2.urlopen, urls)
# Close the pool and wait for the work to finish 
pool.close() 
pool.join() 
```

如果自己去维护一个线程池并且采用队列来完成这个任务会是这样：

```python
'''
A more realistic thread pool example 
'''

import time 
import threading 
import Queue 
import urllib2 

class Consumer(threading.Thread): 
    def __init__(self, queue): 
        threading.Thread.__init__(self)
        self._queue = queue 

    def run(self):
        while True: 
            content = self._queue.get() 
            if isinstance(content, str) and content == 'quit':
                break
            response = urllib2.urlopen(content)
        print 'Bye byes!'


def Producer():
    urls = [
        'http://www.python.org', 'http://www.yahoo.com'
        'http://www.scala.org', 'http://www.google.com'
        # etc.. 
    ]
    queue = Queue.Queue()
    worker_threads = build_worker_pool(queue, 4)
    start_time = time.time()

    # Add the urls to process
    for url in urls: 
        queue.put(url)  
    # Add the poison pill
    for worker in worker_threads:
        queue.put('quit')
    for worker in worker_threads:
        worker.join()

    print 'Done! Time taken: {}'.format(time.time() - start_time)

def build_worker_pool(queue, size):
    workers = []
    for _ in range(size):
        worker = Consumer(queue)
        worker.start() 
        workers.append(worker)
    return workers

if __name__ == '__main__':
    Producer()

```

对比就知道采用`multiprocessing.dummy`的优势了。一般来说，执行 CPU 密集型任务时，调用越多的核速度就越快。但是当处理网络密集型任务时，需要通过实验来确定线程池的大小才是明智的。

### References

* [https://segmentfault.com/a/1190000000414339](https://segmentfault.com/a/1190000000414339)
* [https://pypi.org/project/threadpool/](https://pypi.org/project/threadpool/)
* [https://docs.python.org/3/library/concurrent.futures.html#threadpoolexecutor](https://docs.python.org/3/library/concurrent.futures.html#threadpoolexecutor)



-End-
