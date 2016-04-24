---
layout: post
title: LRU算法的原理和实现
date: 2016-04-23
categories:
- Algorithm
- funny
tags: [funny，Algorithm]
status: publish
type: post
published: true
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---

### LRU 的来源

LRU， Least Recently Used 近期最少使用算法， 常应用于缓存中的数据淘汰，
 其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高“。

### 一般实现思路

仔细看下这个算法的定义： 近期最少使用算法，其实就是按照"近期最少使用"这个条件去淘汰相应的数据。

举个例子，内存的访问速度要远远大于硬盘，所以应该把经常使用的数据放内存，而不常用的数据放磁盘,
假如有一块内存只能存放100个数据，有个程序负责管理这块内存的数据，流程大致如下：

	客户端 -> 请求某个数据 -> 程序处理 -> 内存中有数据 -> 直接返回数据  
								|-> 内存无数据 -> 读取磁盘 -> 存储到内存 -> 返回数据

这样，当不停的访问新的数据，内存迟早会满。

所以就需要一个算法，在内存满的时候，淘汰那些不常用的数据，空出空间存储新的数据，这时可以用LRU。

既然是淘汰最近最少使用的数据，姑且就可以理解为，当内存满了的那个时刻，内存中，哪些数据最后一次被访问的时间最小，
不表示哪些数据是冷数据，应该被移除。

假如每条数据有一个属性lasttime，用来记录被访问时刻的时间，这样，每一条数据都有一个最后访问时间，
当内存满的时候，遍历所有元素，删除最后访问时间最小的那个元素：

    lasttime = current_time()
    lastKey = null;
    if list full:
        foreach each item of list:
            if item.lasttime < lasttime:
                lasttime = item.lasttime
                lastKey = item.key
         
         

然后把lastKey指向的那条数据删除，当然可以改进上面的算法，一次删除多条数据，提高性能。

这个算法是可行的，但是有一个很大的问题，慢！
试想一下，假如有一千万条数据，每次删除都需要找出访问时间最早的那些数据，这是很耗资源的操作，
时间复杂度是O（N），跟数据量成正比，数据量越大,性能越低。

按照这个思路的实现代码（非线程安全）：

[LRUCachePool.java][]


### 优雅的实现思路

上文提到的方法，最大的瓶颈就在于遍历淘汰数据，如何优化这个过程？

细想一下，淘汰的数据，是最后访问时间最早的，上文的思路是为每一条记录加一个lastime，是否有更好的方法，
能够迅速（O(1)复杂度下）找到要淘汰的数据，删除操作也是常量级，性能肯定要高很多。

其实稍微转变一下思路，假如操作的数据在一个链表里面，试想这样一个过程：

    访问一条记录 -> 把链表head 指向当前记录 -> 当前记录中的next指向head之前指向的记录 -> 返回当前数据
    
每访问一次数据，都把最新的访问的数据放到了链表头部，那链表尾部的数据就是最近没有访问过的数据！！
当链表满了，从链表尾部开始往前删除指定数目的数据，就能在常数级时间内腾出空间！

当想到的这个思路的时候，第一感觉，太神奇了，第二感觉，领会到数据结构是如何优雅的解决一些看似复杂耗时的问题！ 

然后看下关键的代码(非线程安全)，首先是记录entry，链表里面的node

~~~java
	public class Entry{
		public Entry prev; 
		public Entry next; 
		public int value;  
		public int key;
		
		public Entry(int value, int key, Entry prev, Entry next){
			this.value  = value;
			this.key 	= key;
			this.prev   = prev;
			this.next   = next;
		}
	}

~~~  

淘汰算法：

~~~java
    // invalid the least used entity
    public void lru(){
    	// here is a way to improve performance, 
    	// if we remove last 10% entries, then this function may called less.
    	
    	// remove the last one
    	Entry entry = this.tail.prev;
    	
    	this.tail.prev = entry.prev;
    	entry.prev.next = this.tail;
    	map.remove(entry.key);
    	this.length--;
    }
~~~

这里只实现的了一次淘汰最后一条记录，当链表满的时候，每插入一条数据，都会调用这个方法删除最后的数据，
性能一般，假如一次删除链表的1%,或者固定数目的多条数据，就能很明显的减少lru方法调用！

当然一次具体淘汰多少数据需要根据具体使用场景来确定，删除太多，可能会引起缓存命中率降低，太少，淘汰操作调用过多，很难给出个经验值。

完整代码：
[LRUCache.java][]

[LRUCachePool.java]: https://github.com/dongyado/awesome-stuff/blob/master/src/top/shares/funny/lru/LRUCachePool.java
[LRUCache.java]: https://github.com/dongyado/awesome-stuff/blob/master/src/top/shares/funny/lru/LRUCache.java