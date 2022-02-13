---
title: 优先权队列
date: 2015-04-19 21:09 +0800
categories: algorithm
---

有这样的应用场景需求：
有一个系统上线有一段时间了，客户针对系统提出了很多bug反馈。这些bug中很多都是相同的，但是提出bug是有先后顺序的。现在要处理这些bug，有如下规则：

* 每次取出bug出现次数最多的bug
* 对于次数相同的bug，优先取出先提出的bug 
* 时间复杂度尽可能低

这个问题可以抽象为用堆实现一个优先权队列，进队列的时候确保该bug的出现次数加一并且在位置得到调整，出队列的时候能找到符合要求的bug并将其移除。

这里用字符串表示bug的描述，用Python的heapq实现该应用场景的要求：
```
import heapq
import itertools

class PriorityQueue:
    def __init__(self):
        self.pq = []                        # list of entries arranged in a heap
        self.entry_finder = {}              # mapping of tasks to entries
        self.REMOVED = '<removed-task>'     # placeholder for a removed task
        self.counter = itertools.count()    # unique sequence count
        
    def add_task(self, task):
        'Add a new task or update the priority of an existing task'
        prior = 1
        if task in self.entry_finder:
            prior = self.entry_finder[task][0]
            prior = -prior + 1
            self.remove_task(task)

        count = next(self.counter)
        entry = [-prior, count, task]
        self.entry_finder[task] = entry
        heapq.heappush(self.pq, entry)

    def remove_task(self, task):
        'Mark an existing task as REMOVED.  Raise KeyError if not found.'
        entry = self.entry_finder.pop(task)
        entry[-1] = self.REMOVED

    def pop_task(self):
        'Remove and return the lowest priority task. Raise KeyError if empty.'
        while self.pq:
            priority, count, task = heapq.heappop(self.pq)
            if task is not self.REMOVED:
                del self.entry_finder[task]
                return task
        raise KeyError('pop from an empty priority queue')

q = PriorityQueue()
q.add_task('foo')
q.add_task('foo')
q.add_task('bar')
q.add_task('spam')
q.add_task('bar')
q.add_task('bar')

print q.pop_task()
print q.pop_task()
print q.pop_task()
```
上述程序中priority其实在这个应用场景下就是bug出现的计数。add和pop的时间复杂度为O(log(N)).
