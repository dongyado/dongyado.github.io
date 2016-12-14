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

本文将分三个部分：

* 源数据转变成ip2region db 文件的过程
* ip2region 的结构
* 搜索方法

### 一. 源数据如何存储到ip2region.db

#### 1. 源数据来源与结构

ip2region 的ip数据来自纯真和淘宝的ip数据库，每次抓取完成之后会生成 ip.merge.txt，
再通过程序根据这个源文件生成ip2region.db 文件。

ip.merge.txt 中每一行对应一条完整记录，每一条记录由ip段和数据组成，格式如下：

    0.0.0.0|0.255.255.255|未分配或者内网IP|0|0|0|0
    1.0.0.0|1.0.0.255|澳大利亚|0|0|0|0
    1.0.1.0|1.0.3.255|中国|华东|福建省|福州市|电信
    1.0.4.0|1.0.7.255|澳大利亚|0|0|0|0
    1.0.8.0|1.0.15.255|中国|华南|广东省|广州市|电信
    1.0.16.0|1.0.31.255|日本|0|0|0|0
    1.0.32.0|1.0.63.255|中国|华南|广东省|广州市|电信
    1.0.64.0|1.0.127.255|日本|0|0|0|0
    1.0.128.0|1.0.255.255|泰国|0|0|0|0
    1.1.0.0|1.1.0.255|中国|华东|福建省|福州市|电信

从左到右分别表示： 起始ip,结束ip,国家，区域，省份，市，运营商。无数据区域默认为0。

最新的ip.merge.txt 有122474条记录，并且根据开始ip地址升序排列。

#### 2. 如何生成ip2region.db 

给定一个ip，如何快速从ip.merge.txt中找到该ip所属记录？最简单的办法就是顺序遍历，当该ip在某条记录起始和结束ip之间时，即命中。

这是低效的做法，如何提高查询性能？用过mysql和其他数据库的的都知道，使用索引。所以ip2region.db使用了内建索引，直接将性能提升到0.0x毫秒级别。

根据ip.merge.txt，为所有数据生成一份索引，并和数据地址组成一个索引项(index block), 然后按起始ip升序排列组成索引，并存储到数据文件的末尾，最终生成的ip2region.db文件大小只有3.5M。

此时的数据库文件中的每一条索引都指向一条对应的数据，也就是说如 

    |中国|华南|广东省|广州市|电信 

这样的数据在文件中被重复存储了很多次，再经过去重优化之后，ip2region.db只有1.5M了，此时把数据库文件全部读取到内存再查找都是非常可行的。

### 二. ip2region.db 结构

生成的ip2region.db文件包含以下四个部分：

1, SUPER BLOCK  
2, HEADER INDEX  
3, DATA    
4, INDEX

![image](/images/post/ip2region.db.png)

生成 ip2region.db 的时候，首先会在首部预留 8 bytes 的SUPER BLOCK 和 8k 的 HEADER INDEX。

再根据ip.merge.txt，依据每一条记录的起始ip, 结束ip和数据，生成一个index block， 前四个字节存储起始ip, 中间四个字节存储结束ip, 后四个字节存储已经计算出的数据地址，并暂存到INDEX区。

当 INDEX 索引区和 DATA 数据区确定下来之后，再把 INDEX 的起始位置存储到 SUPER BLOCK 的前四个字节，结束位置存储到 SUPER BLOCK 的后四个字节。

再把 INDEX 分成大小为 4K 的索引分区，把每个分区起始位置的索引的起始ip和该索引的位置存入一个 header index block, 组成 HEADER INDEX 区域, 最后写入ip2region.db。

具体功能：

* INDEX 

    索引区域，索引元素为 index block (12 字节)， 分成三个部分，起始ip, 结束ip, 数据信息, 每一条 index block 对应 ip.merge.txt 中的一条记录。

    数据信息： 前三个字节保存数据地址（DATA中），后一个字节保存数据长度。

    每个index block 表示一个ip段的索引。当指定ip 在某个 index block 的起始ip和结束ip中间，即表示命中索引。

    再通过 index block 中的数据地址和数据长度，就能从ip2region.db读取对应的地址。

* SUPER BLOCK 
    
    用来保存 INDEX 的起始地址和结束地址，first index ptr 指向INDEX起始位置的index block， last index ptr 指向最后一个index block的地址。这样查询的时候直接读取superblock 8个字节，就能快速获取 INDEX 索引区域的地址。

* HEADER INDEX 
    
    HEADER INDEX 区是对 INDEX 区的二级索引， INDEX总长度除以 4K 就是 HEADER INDEX 的实际索引数。
    
    该区域长度为8k, 由 8 bytes 的 header index block 组成。 
    
    header index block 前四个字节存储每个4K分区起始位置的index block 的起始ip值，后四个字节指向该index block的地址。

    把HEADER INDEX 区定义为8k，可以通过一次磁盘读取读取整个HEADER INDEX 区，然后在内存中进行查询，查询的结果可以确定该ip在INDEX区的某个4k分区内，然后再根据地址一次读取4k index 到内存，再在内存中查询，从而减少磁盘读取的次数。
    

* DATA 

    保存的数据，数据格式如下：

        2163|中国|华南|广东省|深圳市|鹏博士

    分别表示 城市ip,国家，区域，省份，城市，运营商


### 三. 搜索方法

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
                $dataPtr = self::getLong($buffer, 8); // getLong 函数将字节的顺序反过来了
                break;
            }
        }
    }

    // 下面这段代码看起来似乎是，第一个字节存储的长度，后三个字节存储的数据位置
    // 其实是上文的 getLong 函数在获取数据的时候对字节顺序做了一下反转，具体参考 getLong 函数的代码
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
[ip.merge.txt]: https://github.com/lionsoul2014/ip2region/blob/master/data/ip.merge.txt
