---
layout: post
title: Memory Mapped File
date: 2022-02-17 06:00:00 +0800
categories: linux
---
## 内存映射

内存映射是指**磁盘文件**和**进程逻辑地址空间**之间一块大小相同的区域之间的映射。


`mmap`系统调用所做的事情就是建立这样一个映射的关系，并不会把磁盘上文件的内容拷贝到内存。`mmap`函数会在[vm_area_struct](https://elixir.free-electrons.com/linux/v4.15/source/include/linux/mm_types.h#L280)存储一个指向`file`结构的指针。

![](/assets/images/mmap_1.gif)

`mmap`会返回一个指针，指向逻辑地址空间的一个地址，进程通过ptr对文件进行读写操作。但是ptr所指向的是逻辑地址，必须通过MMU转换成物理地址，如上图中过程2。

在这个过程中MMU在地址映射表中是无法找到ptr对应的物理地址的，这样产生一个缺页中断，缺页中断响应函数会去swap中寻找对应的页面，如果找不到则会通过mmap建立的映射关系，从硬盘上将文件内容读取到内存中，如上图过程3。

与之相对的是read的系统调用，需要完成两次数据拷贝。

![](/assets/images/mmap_2.gif)

[Posix doc](https://pubs.opengroup.org/onlinepubs/7908799/xsh/mmap.html)

> `mmap`函数只是给所关联的file descripor增加了一个额外的引用，调用file descritor对应的close方法并不能将这个引用删除。这个引用会在这个文件没有对应的文件映射的时候删除。

如果要删掉对应的映射关系，可以调用函数`munmap`。

## Arrow
Arrow的MemoryMappedFile的实现是使用了一个`Buffer`的子类`Region`，并且是在`Region`的析构函数里面调用的`munmap`, 所以即使对MemoryMappedFile调用了Close方法，只要对应的Buffer还有引用，那么这样的一个映射关系就不会失效。

## References

* [https://medium.com/i0exception/memory-mapped-files-5e083e653b1](https://medium.com/i0exception/memory-mapped-files-5e083e653b1)
* [https://arrow.apache.org/docs/python/memory.html#on-disk-and-memory-mapped-files](https://arrow.apache.org/docs/python/memory.html#on-disk-and-memory-mapped-files)
* [https://stackoverflow.com/questions/55344174/c-close-a-open-file-read-with-mmap](https://stackoverflow.com/questions/55344174/c-close-a-open-file-read-with-mmap)
* [https://blog.csdn.net/mengxingyuanlove/article/details/50986092](https://blog.csdn.net/mengxingyuanlove/article/details/50986092)
