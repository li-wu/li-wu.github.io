---
date: 2017-08-24 09:05
title: Python装饰器
categories: python
---

Python的装饰器有很多应用场景，比如插入日志，性能测试，事务处理，缓存，权限校验等。使用装饰器可以把很多很多和功能无关的代码剥离出来以达到重用的目的。顾名思义，装饰器就是给已有的对象添加额外的功能。

比如，有如下函数say_hello：
```
def say_hello():
    print 'hello'
```
如果要在此函数中添加日志来追踪这个函数被调用时进入函数和退出函数时的事件，可以这么做：
```
def say_hello():
    print 'hello'
    
def print_debug()：
    print 'enter say_hello'
    say_hello()
    print 'exit say_hello'

print_debug()
```
上面的函数输出：
```
enter say_hello
hello
enter say_hello
```
有了装饰器之后，可以用更简单的方式实现：
```
def log_info(fn):
    def new_say_hello():
        print 'enter say_hello'
        fn()
        print 'enter say_hello'
    return new_say_hello

@log_info
def say_hello():
    print "hello"

say_hello()
```
我们其实定义了一个函数log_info，log_info接受一个函数作为参数。同时我们在log_info里面定义了一个新的函数new_say_hello，在new_say_hello中我们打印进入日志，执行传进来的函数参数fn，然后打印退出日志。最后在log_info中将这个定义的函数返回。然后用刚定义的函数log_info来修饰say_hello, 执行say_hello()我们得到了和上面一样的结果。
在new_say_hello的定义中，我们是用了闭包，访问了函数定义外的fn。用log_info来装饰say_hello后，调用say_hello(), 等同于调用log_info(say_hello)()。看一下修饰之后say_hello的函数名：
```
print say_hello.__name__
```
输出：
```
new_say_hello
```
这里得到的却是装饰器中log_info中定义的函数new_say_hello。这可能并非我们想要的效果，我们希望say_hello即使经过装饰之后，得到的依然是原来的函数名，并保留如\_\_doc\_\_之类的元信息。Python的functools包里面提供了wraps可以实现这一点。只需要在新定义的函数new_say_hello上加上@wraps(fn)装饰就可以。
```
from functools import wraps

def log_info(fn):
    @wraps(fn)
    def new_say_hello():
        print 'enter say_hello'
        fn()
        print 'enter say_hello'
    return new_say_hello

@log_info
def say_hello():
    '''
        Say Hello
    '''
    print "hello"

print say_hello.__name__
print say_hello.__doc__
```
输出：
```
say_hello
        Say Hello
```
接着看，如果我们定义的一个函数比如add，接受两个参数，返回它们之和，按照上面的写法就不能达到log的目的。这时我们就需要继续修改上面的装饰器了。在new_add中接受可变参数，并且在调用fn的时候将可变参数传入，同时记住fn执行后的返回值，在new_add最后将结果返回。
```
from functools import wraps

def log_info(fn):
    @wraps(fn)
    def new_add(*args, **kwargs):
        print 'enter add func'
        result = fn(*args, **kwargs)
        print 'exit add func'
        return result
    return new_add
    
@log_info
def add(x, y):
    return x + y

add(1, 2)
```
输出：
```
enter add func
exit add func
3
```
目前看来上面的装饰器已经比较完整了。如果装饰器本身也有参数怎么办？比如上面的log_info能接受一个参数prefix，默认值是'***', 加在所打印的日志之前作为前缀。这时需要在原来的装饰器上包装一个函数来接受参数。
```
from functools import wraps

def log_info(prefix='***'):
    def with_prefix(fn):
        @wraps(fn)
        def new_add(*args, **kwargs):
            print prefix + 'enter add func'
            result = fn(*args, **kwargs)
            print prefix + 'exit add func'
            return result
        return new_add
    return with_prefix

@log_info(prefix='###')
def add(x, y):
    return x + y

print add(1, 2)
```
输出：
```
###enter add func
###exit add func
3
```
上面的函数调用过程是：
log_info(prefix='###') => with_prefix(add) [with closure value prefix='###'] => new_add(1, 2)[with closure value prefix='###'] => add(1, 2)

如果我们定义了多个装饰器，并且应用到同一个函数对象上装饰，会有什么效果?
```
from functools import wraps

def other(fn):
    @wraps(fn)
    def other_decorator(*args, **kwargs):
        print 'Other decorator: enter fn'
        result = fn(*args, **kwargs)
        print 'Other decorator: exit fn'
        return result
    return other_decorator

def log_info(prefix="***"):
    def with_prefix(fn):
        @wraps(fn)
        def new_add(*args, **kwargs):
            print prefix + 'enter add func'
            result = fn(*args, **kwargs)
            print prefix + 'exit add func'
            return result
        return new_add
    return with_prefix

@other
@log_info(prefix="###")
def add(x, y):
    return x + y

print add(1, 2)
```
输出结果：
```
Other decorator: enter fn
###enter add func
###exit add func
Other decorator: exit fn
3
```
如果将两个装饰器的顺序换一下：
```
@log_info(prefix="###")
@other
def add(x, y):
    return x + y

print add(1, 2)
```
输出：
```
###enter add func
Other decorator: enter fn
Other decorator: exit fn
###exit add func
3
```
可以看出下面的装饰器的顺序等效于: f = a(b(c(f)))
```
@a
@b
@c
def f():
    pass
```
最后看一看类的装饰器，相对于函数装饰器，类装饰器灵活度更高，封装性更好。使用类装饰器还可以依靠类内部的\_\_call\_\_方法，当使用 @ 形式将装饰器附加到函数上时，就会调用此方法。
以下是一个简单的示例：
```
class Foo(object):
    def __init__(self, func):
        self._func = func

    def __call__(self):
        print 'enter class decorator'
        self._func()
        print 'exit class decorator'

@Foo
def say_hello():
    print 'Hello'

say_hello()
```
