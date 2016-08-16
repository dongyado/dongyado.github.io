---
layout: post
title: ip2region 数据库文件结构及原理
date: 2015-05-13 12:33:18.000000000 +08:00
categories:
- tools
tags: [tool, ip2region]
status: publish
type: post
published: true
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---

ip2region 是一个准确率99.9%的ip地址定位库。
0.0x毫秒级查询，数据库文件大小只有1.5M，提供了java, php, c, python查询客户端和Binary,B树,内存三种查询算法。

本文将详细介绍ip2region的数据库文件的结构和原理，有兴趣的完全可以依据本文的内容写出自己的查询客户端。

### ip2region.db 结构

如下图:

![image](/images/post/ip2region.db.png)


分为四个部分：

1 SUPER BLOCK

2 HEADER INDEX

3 DATA  

4 INDEX

具体作用：

* SUPER BLOCK 
    
    用来保存 INDEX 的起始地址和结束地址，first index ptr 指向index 头部， last index ptr 指向尾部

* HEADER INDEX 
    
    可以理解为一级索引(btree 搜索用到)，由连续的header index block 组成， header index block 为8个字节， 前四个字节为ip值， 后四个字节为该ip值在 INDEX 中的位置。

    生成数据的时候，把 INDEX 分成 4K的连续分区，header index block 头部保存每个4K分区的第一个ip值，尾部指向该ip值的地址（即该4k分区的起始地址）

* DATA 

    保存的数据

* INDEX 

    索引，索引元素为 index block (12 字节)， 分成三个部分，起始ip, 结束ip, 数据信息。
    数据信息： 头部一个字节保存数据长度，后面三个字节保存数据地址（DATA中）。

### 搜索原理

数据文件的结构是为了让搜索更快，下面介绍一下二分法查找和btree查找，便于大家加深理解。

* binary 原理

    二分法就不多介绍了，直接说步骤

    * 把ip值通过ip2long 转为长整型
    * 通过 SUPER BLOCK 拿到INDEX 的头部和尾部的 index block 
    * 相减+1得出总block数
    * 求出中位数，并拿到中位 index block 
    * 采用二分法直接求解，比较 index block 和当前ip的大小，即可找到该ip属于的 index block
    * 拿到该 index block 的后面四个字节， 分别得到数据长度和数据地址
    * 从数据地址读取拿到的所得长度的字节，即是搜索结果

* b-tree 原理
