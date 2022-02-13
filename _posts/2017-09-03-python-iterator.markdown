---
date: 2017-09-03 06:29 +0800
title: Python迭代器，生成器，协程
categories: python
---

## 迭代器
### 迭代器和可迭代对象
*迭代器(Iterator)*是一个可以遍历容器（特别是列表）的对象。Python中任意对象，只要定义了可以返回一个迭代器的\_\_iter\_\_方法，或者定义了可以支持下标索引的\_\_getitem\_\_方法，那么它就是一个*可迭代对象（Iterable）*。

支持迭代器协议需要实现\_\_iter\_\_和next方法（Python3中\_\_next\_\_），其中\_\_iter\_\_返回迭代器对象，next返回下一个迭代对象，如果迭代结束则会抛出StopIteration异常。

先看一个简单的例子：
```
li = [1,2,3]
it = iter(li)
print it
print it.next()
print it.next()
print it.next()
```
输出结果:
```
<listiterator object at 0x103bc9810>
1
2
Traceback (most recent call last):
  File "iter.py", line 21, in <module>
    print it.next()
StopIteration
```
list对象li就是一个可迭代对象，可以通过内建函数iter()来得到一个迭代器对象it，通过next方法可以访问list中的元素，当没有元素可以访问的时候抛出StopIteration异常。使用for语句的时候，for语句就会自动的通过\_\_iter\_\_()方法来获得迭代器对象，并且通过next()方法来获取下一个元素，遇到 StopIteration 异常时会自动结束迭代。
再来看看string对象：
```
my_string = "Hello World"
next(my_string)
```
输出：
```
TypeError: str object is not an iterator
```
这说明string对象并不是一个迭代器，但是它是一个可迭代对象（dir（my_string）可以看到它实现了\_\_getitem\_\_()方法）。所以可以使用内置函数iter()从该可迭代对象上返回一个迭代器。
```
my_string = "Hello World"
my_iter = iter(my_string)
next(my_iter)
# Output: 'H'
```
### 自定义迭代器
自定义迭代器实际上就是实现一个带\_\_iter\_\_和next方法的类，用该类创建的对象即可迭代对象（因为实现了\_\_iter\_\_方法）。比如要实现一个功能类似xrange的MyRange的迭代器，可以这么写：
```
class MyRange(object):
    def __init__(self, n):
        self.idx = 0
        self.n = n

    def __iter__(self):
        return self

    def next(self):
        if self.idx < self.n:
            val = self.idx
            self.idx += 1
            return val
        else:
            raise StopIteration()
myRange = MyRange(3)
for i in myRange:
    print i
```
可迭代对象即具有 \_\_iter\_\_() 方法的对象，该方法可获取其迭代器对象。迭代器对象即具有 next()方法的对象。**也就是说，一个实现了 \_\_iter\_\_() 方法的对象是可迭代的，一个实现了 next() 方法的对象则是迭代器**。可迭代对象也可以是迭代器对象，如文件对象。此时可迭代对象自己有 next() 方法，而其 \_\_iter\_\_() 方法返回的就是它自己。对于许多内置对象及其派生对象，如 list、dict 等，由于需要支持多次打开迭代器，因此自己并非迭代器对象，需要用 \_\_iter\_\_() 方法返回其迭代器对象，并用迭代器对象来访问其它元素。

上述例子中的 myRange 这个对象就是一个可迭代对象，同时它本身也是一个迭代器对象。对于一个可迭代对象，如果它本身又是一个迭代器对象，就会有这样一个问题，其没有办法支持多次迭代。如下所示：
```
myRange = MyRange(3)
print myRange is iter(myRange)

print [i for i in myRange]
print [i for i in myRange]
```
运行结果：
```
True
[0, 1, 2]
[]
```
为了解决上面的问题，可以分别定义可迭代类型对象和迭代器类型对象；然后可迭代类型对象的 \_\_iter\_\_()方法可以获得一个迭代器类型的对象。如下所示：
```
class Zrange:
    def __init__(self, n):
        self.n = n

    def __iter__(self):
        return ZrangeIterator(self.n)

class ZrangeIterator:
    def __init__(self, n):
        self.i = 0
        self.n = n

    def __iter__(self):
        return self

    def next(self):
        if self.i < self.n:
            i = self.i
            self.i += 1
            return i
        else:
            raise StopIteration()    

zrange = Zrange(3)
print zrange is iter(zrange)         

print [i for i in zrange]
print [i for i in zrange]
```
运行结果：
```
False
[0, 1, 2]
[0, 1, 2]
```

## 生成器
任何包含 yield 语句的函数称为*生成器(generator)*。生成器有如下特点：
* 语法上和函数类似，不一样的是函数用return返回一个值，生成器用yield返回一个值；
* 自动实现迭代器协议，可以调用它的next方法，在没有值返回时抛出StopIteration异常;
* 状态挂起，yield语句一次返回一个结果，并挂起生成器的状态，保留足够的信息以便之后在离开的地方继续执行；

看两个简单的例子：
```
def Zrange(n):
    print "beginning of Zrange"
    i = 0
    while i < n:
        print "before yield", i
        yield i
        i += 1
        print "after yield", i

    print "endding of Zrange"

zrange = Zrange(3)
print "------------"

print zrange.next()
print "------------"

print zrange.next()
print "------------"

print zrange.next()
print "------------"

print zrange.next()
print "------------"
```
运行结果：
```
--------------------
beginning of Zrange
before yield 0
0
--------------------
after yield 1
before yield 1
1
--------------------
after yield 2
before yield 2
2
--------------------
after yield 3
endding of Zrange
Traceback (most recent call last):
  File "one.py", line 38, in <module>
    print zrange.next()
StopIteration
```
通过运行结果可以看出：
* 当调用生成器函数的时候，函数只是返回了一个生成器对象，并没有执行；
* 当next()方法第一次被调用的时候，生成器函数才开始执行，执行到yield语句处停止；
* next()方法的返回值就是yield语句处的参数（yielded value）；
* 当继续调用next()方法的时候，函数将接着上一次停止的yield语句处继续执行，并到下一个yield处停止；如果后面没有yield就抛出StopIteration异常；

## 列表解析和生成器表达式
### 列表解析
列表解析语法结构：
```
[expr for iter_var in iterable if cond_expr]
```
例如用 lambda 函数计算序列成员的平方的表达是为：
```
map(lambda x: x ** 2, range(6))
```
改用列表解析来写更加清晰：
```
[x ** 2 for x in range(6)]
```
### 生成器表达式
生成器表达式是列表解析的一个扩展。列表解析的一个不足就是必要生成所有的数据, 用以创建整个列表。这可能对有大量数据的迭代器有负面效应。生成器表达式通过结合列表解析和生成器解决了这个问题。生成器表达式在 Python 2.4 被引入, 它与列表解析非常相似，而且它们的基本语法基本相同; 不过它并不真正创建数字列表, 而是返回一个生成器，这个生成器在每次计算出一个条目后，把这个条目“产生”(yield)出来。生成器表达式使用了”延迟计算”(lazy evaluation), 所以它在使用内存上更有效。生成器表达式语法:
```
(expr for iter_var in iterable if cond_expr)
```
## 协程
Python中协程和生成器相似但是又不同。主要区别是：
* 生成器是数据的生产者
* 协程是数据的消费者

下面是Python实现的grep的一个例子：
```
def grep(pattern):
    print "Searching for %s" % pattern
    while True:
        line = (yield)
        if pattern in line:
            print line
```
从上面例子可以看出yield并没有返回值，相反要从外部传值给它。可以通过send()方法向它传值。
```
search = grep('coroutine')
next(search)
#output: Searching for coroutine
search.send("I love you")
search.send("Don't you love me?")
search.send("I love coroutine instead!")
#output: I love coroutine instead!
```
发送的值会被yield接受，为啥要运行next()方法呢？这样做是为了启动一个协程。就像协程中包含的生成器不会立刻执行，而需要通过next()方法来响应send()方法。因此必须通过next()方法来执行yield表达式。
可以通过close()方法来关闭协程：
```
search = grep('coroutine')
search.close()
```
