---
layout: post
title: ip2region 数据库文件结构及原理
date: 2016-08-18
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

[ip2region][] 是一个准确率99.9%的ip地址定位库。
0.0x毫秒级查询，数据库文件大小只有1.5M，提供了java, php, c, python查询客户端和Binary,B树,内存三种查询算法。

本文将详细介绍ip2region的数据库文件的结构和原理，有兴趣的童鞋可以依据本文的内容写出自己的查询客户端。

### 一. ip2region.db 结构

如下图，有四个组成部分：

1, SUPER BLOCK  
2, HEADER INDEX  
3, DATA    
4, INDEX

![image](/images/post/ip2region.db.png)

具体功能：

* SUPER BLOCK 
    
    用来保存 INDEX 的起始地址和结束地址，first index ptr 指向index 头部， last index ptr 指向尾部

* HEADER INDEX 
    
    可以理解为一级索引(btree 搜索用到)，由连续的header index block 组成， header index block 为8个字节， 前四个字节为ip值， 后四个字节为该ip值在 INDEX 中的位置。

    生成数据的时候，把 INDEX 分成 4K的连续分区，header index block 头部保存每个4K分区的第一个ip值，尾部指向该ip值的地址（即该4k分区的起始地址）

* DATA 

    保存的数据，数据格式如下：

        2163|中国|华南|广东省|深圳市|鹏博士

    分别表示 城市ip,国家，区域，省份，城市，运营商

* INDEX 

    索引，索引元素为 index block (12 字节)， 分成三个部分，起始ip, 结束ip, 数据信息。
    数据信息： 头部一个字节保存数据长度，后面三个字节保存数据地址（DATA中）。

    每个index block 表示一个ip段的索引。当指定ip 在某个index block 的起始ip和结束ip中间，即表示命中索引。

### 二. 搜索方法

数据文件的结构是为了让搜索更快，下面介绍一下二分法查找和btree查找，便于大家加深理解。

#### binary搜索

二分法就不多介绍了，步骤：

* 把ip值通过ip2long方法转为长整型
* 通过 SUPER BLOCK 拿到INDEX的起始位置和结束位置 
* 相减+1得出index block 总数
* 采用二分法直接求解，比较 index block 和当前ip的大小，即可找到该ip属于的 index block
* 拿到该 index block 的后面四个字节， 分别得到数据长度和数据地址
* 从数据地址读取拿到的所得长度的字节，即是搜索结果

以php客户端作为例子注释:

~~~php
<?php
    fseek($this->dbFileHandler, 0); 
    $superBlock = fread($this->dbFileHandler, 8); // 从文件0位置往后读取8字节，即 super block
    $this->firstIndexPtr = self::getLong($superBlock, 0); // 获取INDEX起始位置
    $this->lastIndexPtr  = self::getLong($superBlock, 4); // 获取INDEX结束位置
    $this->totalBlocks   = ($this->lastIndexPtr-$this->firstIndexPtr)/INDEX_BLOCK_LENGTH + 1; // 计算总索引数，即 index block 总数

    // 二分法搜索
    $l = 0; // 低位
    $h = $this->totalBlocks; // 高位
    $dataPtr = 0;
    while ( $l <= $h  ) {
        $m = (($l + $h) >> 1); // 中位
        $p = $m * INDEX_BLOCK_LENGTH;
        fseek($this->dbFileHandler, $this->firstIndexPtr + $p); // 移动读取位置
        $buffer = fread($this->dbFileHandler, INDEX_BLOCK_LENGTH); // 读取 INDEX_BLOCK_LENGTH 个字节 （12 字节）, 即读取一个index block
        $sip    = self::getLong($buffer, 0); // 获取开始 ip
        
        // 进行比较
        if ( $ip < $sip  ) {
            $h = $m - 1; // 比中位index block 开始ip小
        } else {
            $eip = self::getLong($buffer, 4);
            if ( $ip > $eip  ) {
                $l = $m + 1; // 比中位index block 的结束ip小
            } else { // 命中数据
                $dataPtr = self::getLong($buffer, 8);
                break;
            }
        }
    }

    // 读取数据
    $dataLen = (($dataPtr >> 24) & 0xFF); // 数据长度
    $dataPtr = ($dataPtr & 0x00FFFFFF);   // 数据位置

    return array(
            'city_id' => self::getLong($this->dbBinStr, $dataPtr),  // 获取城市id
            'region'  => substr($this->dbBinStr, $dataPtr + 4, $dataLen - 4) // 获取其他数据
    );
?>
~~~
    
源码请查阅 [ip2region php client][] 的 binarySearch 方法。

#### b-tree 搜索

b-tree 搜索用到了 HEADER INDEX，第一步先在 HEADER INDEX 中搜索，再定位到 INDEX 中的某个 4k index分区搜索。

步骤： 

* 把ip值通过ip2long 转为长整型
* 使用二分法在 HEADER INDEX 中搜索，比较得到对应的 header index block 
* header index block 指向 INDEX 中的一个 4K 分区，所以直接把搜索范围降低到 4K
* 采用二分法在获取到的 4K 分区搜索，得到对应的 index block
* 拿到该 index block 的后面四个字节， 分别得到数据长度和数据地址
* 从数据地址读取拿到的所得长度的字节，即是搜索结果


具体源码请查阅 [ip2region php client][] 中的 btreeSearch 方法。

### 相关链接

项目地址： https://github.com/lionsoul2014/ip2region


[ip2region]: https://github.com/lionsoul2014/ip2region
[ip2region php client]: https://github.com/lionsoul2014/ip2region/blob/master/binding/php/Ip2Region.class.php
