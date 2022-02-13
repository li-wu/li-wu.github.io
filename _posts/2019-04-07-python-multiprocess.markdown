---
title: Python Multiprocessing Best Practice
date: 2019-04-07
categories: Python
---

GIL的存在使Python中多线程无法充分利用计算机多核的优势来提高性能。对于计算密集型的应用，应该使用多线程来提高性能，Python中提供了multiprocessing模块来处理进程的创建和他们之间的通信和协调。进程不同于线程，每个进程都有其独立的地址空间，所以进程间的数据共享和传递没有[线程间通信](https://python3-cookbook.readthedocs.io/zh_CN/latest/c12/p03_communicating_between_threads.html)那么方便。线程间通信方式主要有两种：Queue和Event。进程间通信主要利用Pipe和Queue，还有用于共享资源的multiprocessing.Value, multiprocessing.Array和Manager等。

multiprocessing模块在使用的时候需要注意如下几点：

### 1. 进程间通信优先考虑Pipe和Queue

进程间通信优先考虑Pipe和Queue，而不是Lock, Event, Condition, Semaphore等同步原语。进程中的类Queue使用pipe和一些locks/semaphore原语来实现，是进程安全的。该类和线程中的Queue基本类似，除了方法task_done()和join()是在子类JoinableQueue中实现以外。由于Queue底层使用pipe来实现，使用Queue来进行进程通信时，传输的对象必须是可以序列化的，否则put操作会导致PicklingError。

在使用Pipe的时候要注意Pipe不支持进程安全，当有多个进程对管道的一端进行读操作或者写操作的时候可能会导致数据丢失或者损坏。如果有超过两个以上进程，可以使用Queue，但是对于两个进程之间的通信Pipe性能更好，这里有一个[benchmark](https://stackoverflow.com/questions/8463008/multiprocessing-pipe-vs-queue)，其中还提到了JoinableQueue的性能。

### 2. 尽量避免资源的共享

相对于线程，进程间资源共享开销比较大，因此要尽量避免使用资源共享。如果无法避免，可以使用multiprocessing.Value和multiprocessing.Array或者multiprocessing.sharedctypes来实现内存共享，也可以通过服务器进程管理器Manager()来实现数据和状态共享。内存共享方式更快，效率高，但是Manager()使用起来更方便。

```python
import time
from multiprocessing import Process, Value

def func(val):
    for i in range(10):
        time.sleep(0.1)
        val.value += 1
        
if __name__ == '__main__':
    v = Value('i', 0)
    processList = [Process(target=func, args=(v,)) for i in range(10)]
    for p in processList:
        p.start()
    for p in processList:
        p.join()
    print v.value
```

注意上面的程序最后的输出并非100。虽然Value的构造函数multiprocessing.Value(typecode_or_type, *args[,lock])中如果lock为True会创建一个锁对象用于同步访问控制，但实际要真正控制同步访问，需要实现获取这个锁，因此需要将`func`修改为：

```python
def func(val):
    for i in range(10):
        time.sleep(0.1)
        with val.get_lock():
            val.value += 1
```

使用Manager()进行内存共享：

```python
import multiprocessing

def f(ns):
    ns.x.append(1)
    ns.y.append('a')
    
if __name__ == '__main__':
    manager = multiprocessing.Manager()
    ns = manager.Namespace()
    ns.x = []
    ns.y = []
    print 'before process operation:', ns
    p = multiprocessing.Process(target=f, args=(ns,))
    p.start()
    p.join()
    print 'after procss operation: ', ns
```

期望的程序输出是x = [1], y = ['a']，但实际是 x = [], y = []。因为manager对象只能传播对一个可变对象本身做的修改。如有一个manager.list()对象，管理列表本身的任何更改会传播到所有其他进程。但是如果容器对象内部还包括可修改的对象，则内部可修改对象的任何更改都不回传播到其它进程。因此正确的方式应该是：

```python
import multiprocessing

def f(ns, x, y):
    x.append(1)
    y.append('a')
    ns.x = x
    ns.y = y
    
if __name__ == '__main__':
    manager = multiprocessing.Manager()
    ns = manager.Namespace()
    ns.x = []
    ns.y = []
    print 'before process operation:', ns
    p = multiprocessing.Process(target=f, args=(ns, ns.x, ns.y))
    p.start()
    p.join()
    print 'after procss operation: ', ns
```

### 3. 注意平台的差异

Linux使用fork()来创建进程，因此父进程中的所有资源包括数据结构、打开的文件或者数据库的连接都会在子进程中共享。而Windows平台子父进程相互独立，因为为了保持平台的兼容性，最好能将相关资源对象作为子进程的构造函数的参数传递进去。避免下面这样的写法：

```python
from multiprocessing import Process
f = None

def child():
    print f

if __name__ == '__main__':
    f = open('mp.py', 'r')                                                      
    p = Process(target=child)
    p.start()
    p.join()
```

而推荐使用如下方式：

```python
from multiprocessing import Process
f = None

def child():
    print f

if __name__ == '__main__':
    f = open('mp.py', 'r')
    # pass resource object as args
    p = Process(target=child, args=(f,))
    p.start()
    p.join()
```

### 4. 避免使用terminate()终止进程

> Using the [`Process.terminate`](https://docs.python.org/2/library/multiprocessing.html#multiprocessing.Process.terminate) method to stop a process is liable to cause any shared resources (such as locks, semaphores, pipes and queues) currently being used by the process to become broken or unavailable to other processes.
>
> Therefore it is probably best to only consider using [`Process.terminate`](https://docs.python.org/2/library/multiprocessing.html#multiprocessing.Process.terminate) on processes which never use any shared resources.

简单来说只有在确保当前要终止的进程没有使用任何共享资源的情况下使用`terminate`来终止进程。

### References

* [https://docs.python.org/2/library/multiprocessing.html#programming-guidelines](https://docs.python.org/2/library/multiprocessing.html#programming-guidelines)
* [https://stackoverflow.com/questions/8463008/multiprocessing-pipe-vs-queue](https://stackoverflow.com/questions/8463008/multiprocessing-pipe-vs-queue)
* [《编写高质量代码：改善Python程序的91个建议》](https://book.douban.com/subject/25910544/)
