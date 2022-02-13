---
title: Sequential Consistency and Release Consistency
date: 2013-03-21 21:53:21 +0800
categories: algorithm
---
一致性是分布式系统中比较让人头痛的问题，原因有如下：  
1.缓存中的数据副本；  
2.并发操作，没有共享的时钟；  
3.机器或者网络错误；  

严格一致性模型(strict consistency)规定:所有的读操作都得到最新的写操作的值;因为没有全局的时钟存在，所以严格一致性模型是不现实的。而下面要说的是顺序一致性(sequential consistency)和释放一致性(release consistency)。  

## Sequential Consistency 

“*the result of any execution is the same as if the operations of all the processors were executed in some sequential order, and the operations of each individual processor appear in this sequence in the order specified by its program*”，即任何一次执行的结果都像所有处理器的操作以某种顺序的次序执行所得到的一样，而且各处理器的操作都按照各自程序所指定的次序出现在这个序列中。  

当程序在各个机器上并行运行时，任何一种有效的交错存储器访问顺序都是可认可的行为，但所有处理器必须看见的是同样的访问顺序。如果一个进程（处理器）看见的是一种交错，另一进程看见的是另一种交错，则这样的存储器不是一个顺序一致性的存储器。然而，同一程序再次并行运行，其存储器访问的交错次序会不同于上次的交错次序，这是允许的。在一块存储器中，若一个进程（处理器）看到一种交错，另一进程看到另一个交错，这就不是顺序一致存储器。  

顺序一致性的典型实现是IVY，中心思想是采用了一个中心管理者，维护了页表的副本，所有者，访问权限等信息，最近的一个写操作者是页表的拥有者。读写操作都要经过中心管理者。每一个页表都只有当前一个拥有者，当前拥有者有这个页表的副本，如果拥有者的权限是r/w,那么就没有其它副本；如果拥有者的权限是r/o,那么拥有者的副本和其它副本是一样的。中心管理者知道所有的副本信息。  

## Release Consistency
顺序一致性一个很大的问题就是false sharing,简单来说就是两个进程同时要修改一个页表里面的两个不同变量，这样的操作却要进行同步，从而引起“Ping-pong”的现象。
释放一致性提供两个操作acquire/release来区分进入和离开临界区。同步协议分为两种：update-base protocol和invalidate-base protocol。失效协议发送的信息更少一些，但是有可能因为数据缺失问题开销变大；更新协议就是将所有的更改信息在release操作之后发送到所有缓存进程中。  

释放一致性分为eager和lazy两种，前一种是主动发消息，后一种是有请求的时候再发消息。

在多个写操作协议里面，可以通过twining方法来检测更改，即在写操作更改页表之前创建一个页表副本，更改操作完成之后做一个diff操作，然后将结果和另外的进程同步merge，这样不仅使得多个写操作同步进行，还减少了信息的交流，同样false sharing的问题也不存在。  

最后要说的是vector timestamps，两张图比较能说明问题：  

![Image Title](/assets/images/23-25-34.jpg)  

![Image Title](/assets/images/23-25-45.jpg)  





