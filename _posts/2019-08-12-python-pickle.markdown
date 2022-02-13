---
title: Multiprocess Pickle Error
date: 2019-08-12
categories: Python
---

最近在把一个项目迁移到Python3，对于`multiprocess`这一部分一直有一个`Pickle Error`困扰了我一天多的时间，这里记录一下。

因为用到了多进程，进程间的通信用的是`JoinableQueue`，而加入队列的对象是能够pickle的才行。 程序在Python2下是没问题的，迁移到Python3之后的错误信息是：

```shell
typeerror cannot serialize '_io.textiowrapper' object
```

Google结果能找到一些类似的错误：

* [https://stackoverflow.com/questions/52225003/writing-to-multiple-files-using-multiprocessing-error-typeerror-cannot-seria](https://stackoverflow.com/questions/52225003/writing-to-multiple-files-using-multiprocessing-error-typeerror-cannot-seria)
* [https://github.com/hyperopt/hyperopt-sklearn/issues/74](https://github.com/hyperopt/hyperopt-sklearn/issues/74)
* [http://databasefaq.com/index.php/answer/114012/python-windows-multiprocessing-cherrypy-python-multiprocessing-multiprocessing-works-in-ubuntu-doesnt-in-windows](http://databasefaq.com/index.php/answer/114012/python-windows-multiprocessing-cherrypy-python-multiprocessing-multiprocessing-works-in-ubuntu-doesnt-in-windows)
* [https://www.oipapio.com/question-2484266](https://www.oipapio.com/question-2484266)

核心大概是放进`Queue`里面的对象必须是能Pickle的。所以应该去检查放进队列里的对象，开始的时候用了`drill`这个package去看对象[是否可以Pickle](https://stackoverflow.com/questions/17872056/how-to-check-if-an-object-is-pickleable)，但是这个并不准确，导致我花了很多时间从其它地方去看这个问题。最后还是得一个个排查每个对象。大致要注意：

* 对象里面不能有`logger`
* 不能有File Handler
* 不能有如socket connection之类的
* 不能有lambda

文档里面有讲什么能pickle什么不能：[https://docs.python.org/3/library/pickle.html#what-can-be-pickled-and-unpickled](https://docs.python.org/3/library/pickle.html#what-can-be-pickled-and-unpickled)

最后在对象的嵌套对象里面找到了一个属性是file handler，大致是这么写的：

```python
def _open_file(self, file):
    self.fh = open(file, 'r')
  
def _close_file(self)
    self.fh.close()
```

这样的对象因为包含`self.fh`就没办法pickle, 导致上面的错误。重写这段代码上述问题就没有了。

有时候真不能太相信第三方包，还是要耐心一点点打日志，调试，排除和定位问题在哪里。程序员就是这样一个职业，每天都会碰到很多问题，然后一点点定位问题，拿出解决问题方案，尝试解决问题，回归测试，检查是否引入额外的问题，每一步都得严谨谨慎，不然心里也不踏实。只是有时候遇到棘手一些的问题并且尝试了很长时间都没有找到问题会沮丧。没办法，这就是职业特点，只能接受并耐心对待，除非不吃这口饭。 

-End-
