---
title: Mac下模拟慢磁盘测试场景
date: 2023-11-30
categories: mac
---

开发过程中有时候要测试慢磁盘的场景，Mac下自带了一个叫做dmc ([disk mount conditioner](https://manp.gs/mac/1/dmc))的命令，可以把一个目录下面的文件访问降速从而达到模拟一个慢速设备的目的。

它预置了一些设备参数可以选择：
```
❯ dmc list
  0: Faulty 5400 HDD
  1: 5400 HDD
  2: 7200 HDD
  3: Slow SSD
  4: SATA II SSD
  5: SATA III SSD
  6: PCIe 2 SSD
  7: PCIe 3 SSD

```
假设我们要限制文件夹`~/data`下的读写速度，那么可以运行如下命令：
```
sudo dmc start ~/data/ "Slow SSD"
```
可以查看`Slow SSD`对应的具体读写速度限制：
```
> dmc show "Slow SSD"
Profile: Slow SSD
 Type: SSD
 Access time: 100 us
 Read throughput: 250 MB/s
 Write throughput: 125 MB/s
 I/O Queue Depth: 32
 Max Read Bytes: 33554432
 Max Write Bytes: 33554432
 Max Read Segments: 64
 Max Write Segments: 64
```

测试完了之后可以使用`dmc stop`来清除之前的设置，使对应文件夹下的读写速度恢复正常。
