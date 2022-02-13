---
title: Python Logging的一点思考
date: 2019-07-22
categories: Python
---

最近在项目中重构Python Logging相关的逻辑。为什么要重构？

* 大量重复代码，为每个Logger设置`handler`, `formatter`, `logLevel`等

* 将`logger`设置为类的成员变量，因为没法序列化，又手动在`__getstate__`里面将`logger`删掉，然后在`__setstate__`里面重新`setup`

  ```python
  # loggers can't be pickled due to the lock object, remove them before we try to pickle anything.
  def __getstate__(self):
      temp = self.__dict__
      if getattr(self, 'logger', None):
        temp.pop('logger', None)
        return temp
  
  def __setstate__(self, d):
      self.__dict__ = d
      self._setup_logging()
  ```

* 日志写得非常随意，遇到问题看日志比较难定位问题在哪里

* 对于多进程的日志处理使用了一个很老的`logutils`的库，很久都没有维护了

### dictConfig()

使用`dictConfig()`来设置`logging`能更清晰，所有的`formatter`, `filter`, `handler`和`logger`都可以同时设置在一起，这里是一个来自[《Logging Cookbook》](https://docs.python.org/3/howto/logging-cookbook.html#configuring-filters-with-dictconfig)的示例：

```python
import logging
import logging.config
import sys

class MyFilter(logging.Filter):
    def __init__(self, param=None):
        self.param = param

    def filter(self, record):
        if self.param is None:
            allow = True
        else:
            allow = self.param not in record.msg
        if allow:
            record.msg = 'changed: ' + record.msg
        return allow

LOGGING = {
    'version': 1,
    'filters': {
        'myfilter': {
            '()': MyFilter,
            'param': 'noshow',
        }
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'filters': ['myfilter']
        }
    },
    'root': {
        'level': 'DEBUG',
        'handlers': ['console']
    },
}

if __name__ == '__main__':
    logging.config.dictConfig(LOGGING)
    logging.debug('hello')
    logging.debug('hello - noshow')
```

同时在在该module里面直接定义对应的`logger`, 使用的时候直接`import`。因为Python模块本身就是单例模式，所以同一个logger实例能被多个模块复用，而不用每次重新去设置一个新的logger。

### structlog

通常我们是这样使用`logging`的：

```python
import logging

log = logging.getLogger()
command = "test"
log.info("Main program started with args: command = {}".format(command))
```

每次这么写会很麻烦，需要把log信息拼接起来，而且在调试的时候也不太容易从日志里面找到对应有用的信息。所以这里我使用了[structlog](http://www.structlog.org/)，简单来讲就是使打出来的日志格式更加可读，并且打日志的时候写起来也更加轻松。使用了`structlog`后就可以这么写上面的例子了：

```python
import structlog
log = structlog.getLogger() # or structlog.get_logger()

log.info("Main program started", command="test")
# 2018-08-23 17:59.51 Main program started                    command=test
```

除了简单易用之外，structlog还提供了data binding/pipelines/formatting/output等强大功能。文档还比较详细，使用场景都有例子说明。

### Logging from Multi Processing

`logging`是线程安全，但并非线程安全的。所以针对多进程的日志处理，要做一些额外的工作。比较推荐的做法是在主进程中起一个线程来处理日志，同时使用`Queue`和`QueueHandler`来处理各个进程中的日志。下面是一个示例程序：

```python
import logging
import logging.config
import logging.handlers
from multiprocessing import Process, Queue
import random
import threading
import time

def logger_thread(q):
    while True:
        record = q.get()
        if record is None:
            break
        logger = logging.getLogger(record.name)
        logger.handle(record)


def worker_process(q):
    qh = logging.handlers.QueueHandler(q)
    root = logging.getLogger()
    root.setLevel(logging.DEBUG)
    root.addHandler(qh)
    levels = [logging.DEBUG, logging.INFO, logging.WARNING, logging.ERROR,
              logging.CRITICAL]
    loggers = ['foo', 'foo.bar', 'foo.bar.baz',
               'spam', 'spam.ham', 'spam.ham.eggs']
    for i in range(100):
        lvl = random.choice(levels)
        logger = logging.getLogger(random.choice(loggers))
        logger.log(lvl, 'Message no. %d', i)

if __name__ == '__main__':
    q = Queue()
    d = {
        'version': 1,
        'formatters': {
            'detailed': {
                'class': 'logging.Formatter',
                'format': '%(asctime)s %(name)-15s %(levelname)-8s %(processName)-10s %(message)s'
            }
        },
        'handlers': {
            'console': {
                'class': 'logging.StreamHandler',
                'level': 'INFO',
            },
            'file': {
                'class': 'logging.FileHandler',
                'filename': 'mplog.log',
                'mode': 'w',
                'formatter': 'detailed',
            },
            'foofile': {
                'class': 'logging.FileHandler',
                'filename': 'mplog-foo.log',
                'mode': 'w',
                'formatter': 'detailed',
            },
            'errors': {
                'class': 'logging.FileHandler',
                'filename': 'mplog-errors.log',
                'mode': 'w',
                'level': 'ERROR',
                'formatter': 'detailed',
            },
        },
        'loggers': {
            'foo': {
                'handlers': ['foofile']
            }
        },
        'root': {
            'level': 'DEBUG',
            'handlers': ['console', 'file', 'errors']
        },
    }
    workers = []
    for i in range(5):
        wp = Process(target=worker_process, name='worker %d' % (i + 1), args=(q,))
        workers.append(wp)
        wp.start()
    logging.config.dictConfig(d)
    lp = threading.Thread(target=logger_thread, args=(q,))
    lp.start()
    # At this point, the main process could do some useful work of its own
    # Once it's done that, it can wait for the workers to terminate...
    for wp in workers:
        wp.join()
    # And now tell the logging thread to finish up, too
    q.put(None)
    lp.join()
```

### References

* [https://docs.python.org/3/howto/logging.html](https://docs.python.org/3/howto/logging.html)
* [https://docs.python.org/3/howto/logging-cookbook.html#configuring-filters-with-dictconfig](https://docs.python.org/3/howto/logging-cookbook.html#configuring-filters-with-dictconfig)
* [https://codywilbourn.com/2018/08/23/playing-with-python-structured-logs/](https://codywilbourn.com/2018/08/23/playing-with-python-structured-logs/)
* [https://docs.python.org/3/howto/logging-cookbook.html#logging-to-a-single-file-from-multiple-processes](https://docs.python.org/3/howto/logging-cookbook.html#logging-to-a-single-file-from-multiple-processes)
* [https://geshan.com.np/blog/2019/03/follow-these-logging-best-practices-to-get-the-most-out-of-application-level-logging-slides/](https://geshan.com.np/blog/2019/03/follow-these-logging-best-practices-to-get-the-most-out-of-application-level-logging-slides/)

