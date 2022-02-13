---
date: 2018-03-16 15:04
title: Python中的字符编码
categories: Python
---

最近在写爬虫的时候遇到了字符编码的一些问题，所以就把Python中字符编码的问题稍微梳理总结一下。

Python 2中有两种表示字符序列的类型， 分别是str和unicode，str实例包含原始的8位值（二进制），而unicode实例则包含Unicode字符。Python 3中表示字符序列的类型是bytes和str，分别表示原始的8位值（二进制）和Unicode字符。Python 2中的unicode实例和Python 3中的str实例都没有和特定的二进制编码形式相关联，如果要将Unicode字符转换为二进制，必须使用encode方法，要把二进制转换为Unicode字符必须使用decode方法。因为 Python 认为 16 位的 unicode 才是字符的唯一内码，而大家常用的字符集如 gb2312，utf-8，以及 ascii 都是字符的二进制（字节）编码形式。把字符从 unicode 转换成二进制编码，要用 encode方法。

把Unicode字符表示成二进制数据有很多种编码方式，最常见的是UTF-8。写程序的最佳实践是把编码和解码工作放在最外围来做，程序的核心使用Unicode字符（Python 2中的unicode和Python 3中的str），而且不要对字符编码做任何假设。这样做能保证程序能接受不同类型的文本编码，又能保证输出的文本只采用一种编码形式，比如UTF-8。

所以在开发的过程中经常会涉及到两种类型的转换，使得转换后的数据符合程序对输入数据的预期。比如在Python 2中将str或者unicode转换为unicode：
```python
def to_unicode(unicode_or_str):
    if isinstance(unicode_or_str, str):
        value = unicode_or_str.decode('utf-8')
    else:
        value = unicode_or_str
    return value  # instance of unicode
```
```python
def to_str(unicode_or_str):
    if isinstance(unicode_or_str, unicode):
        value = unicode_or_str.encode('utf-8')
    else:
        value = unicode_or_str
    return value  # instance of str
```

另外一个需要注意的问题是在读写文件的时候Python 3和Python 2的差异。
```python
with open('/tmp/random.bin', 'w') as f:
    f.write(os.urandom(10))
```
这段代码在Python 3中会出现异常，因为Python 3给open函数增加了名为encoding的新参数，默认值为utf-8，这样在读写操作的时候系统就需要开发者必须传入包含Unicode字符的str实例，而不接受包含二进制的bytes实例。为了统一这种兼容性的问题，按照下面的方式开启二进制模式可以同时适配Python 2和Python 3：
```python
with open('/tmp/random.bin', 'wb') as f:
    f.write(os.urandom(10))
```

再比如如果开发中涉及到中文，采用同样的策略来处理（示例采用Python 2）。
```python
 # -*- coding: utf-8 -*-
import string

# 这个是 str 的字符串
s = '你好世界'

# 这个是 unicode 的字符串
u = u'你好世界'

print isinstance(s, str)      # True
print isinstance(u, unicode)  # True

print s.__class__   # <type 'str'>
print u.__class__   # <type 'unicode'>
```

参考：

* 《Effective Python》
* [http://in355hz.iteye.com/blog/1860787](http://in355hz.iteye.com/blog/1860787)

-End-
