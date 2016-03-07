---
layout: post
title: A*寻路的原理和实现 
date: 2015-11-23 09:18.000000000 +08:00
categories:
- Algorithm
tags: [Algorithm]
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  views: '1'
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
  first_name: ''
  last_name: ''
description: 'A*寻路算法是一个游戏开发讨论最多，而且相当有趣的一个算法'
---
平时很少玩游戏，玩的最多的可能是老掉牙的CS和求生之路， 都是一些FPS游戏，沉迷于打打杀杀的刺激。一直知道游戏的开发是比较复杂的，也不曾去深究。

<!-- more -->

前段时间买了个WINDOWS平板，里面的扫雷很不错，所以玩的比较多，其中有个很奇妙的现象，那个扫雷的小人可以从障碍中（可能只有一个出口）走出来到鼠标点击的地方，想到这其中肯定应用了寻路算法。

请教了一下谷歌娘，发现讨论了最多的就是A*算法，初步看了一下，觉得很有趣，就添加到了TODO list里面，准备花点时间学习一下。

### 什么是A*算法
A*（A-Star)算法是一种静态路网中求解最短路径最有效的直接搜索方法。在包含各种障碍物的地图中，为游戏角色的移动，寻找一条到目标地点最短路径。这个算法不同于A算法，是因为使用了一个启发函数，用来趋势每一步的走向。

### A*算法原理
本文使用了那套最经典的图（来自[A* Pathfinding for Beginners](http://homepages.abdn.ac.uk/f.guerin/pages/teaching/CS1013/practicals/aStarTutorial.htm)，感谢原作者的制作)

![image](http://pic002.cnblogs.com/images/2011/70278/2011052614224354.jpg)

游戏中的地图，都可以看成一个方块集合。上图A表示起点，B表示终点，蓝色方块表示障碍物，我们需要找出一条路，从A点开始，绕过蓝色方块，到达B点。

#### 寻路步骤
首先说一下用到的名词：

* 开放列表：记录没有检查过的方格
* 关闭列表：记录已经检查过的方格 

步骤

    1. 把A放入开放列表
    2. 找出从A点开始移动一步能到达的方块并加入到开放列表，就是环绕A点的方块，如果是障碍物就不加入开放列表
    3. 把A点从开放列表删除，并加入关闭列表，因为A点已经被检查过了 

![image](http://pic002.cnblogs.com/images/2011/70278/2011052522372596.jpg) 

上图绿色边框的方块表示已经加入开发列表，等待下一次检查；中间淡蓝色边框则表示已经被加入关闭列表，不需要再次检查。

然后进入下一步， 从开放列表中找出下一步的要走的方格，那怎么确定下一步要走的方格？

这就是A*算法的核心部分，使用了一个函数引导每一步的走向：

    F=G+H

    G 表示从起点方格移动到网格上指定方格的移动耗费 (可沿斜方向移动).
    H 表示从指定的方格移动到终点方格的预计耗费 (H启发函数).

为了便于计算，向上下左右移动的代价（函数中的G）为10, 向斜线方向移动为14 (其实就是对10^2+10^2之和开平方)。
而函数中的H，采用的是曼哈顿距离，表示两点之间分别在X，Y轴方向之差的和。

所以在一个平面上，坐标（x1, y1）的i点与坐标（x2, y2）的j点的曼哈顿距离为：d(i,j)=|X1-X2|+|Y1-Y2|。

很容易知道，P（3,4）和P（5,3）的曼哈顿距离为H=|5-3| + |3-4| = 3

![image](http://homepages.abdn.ac.uk/f.guerin/pages/teaching/CS1013/practicals/aStarTutorial_files/aStarT3.jpg)

如上图，每个方块的左上角表示F值，左下角表示G值，右下角表示H值。

以绿色背景方块为起点，移动到其周围的方块的G值为10或14。

至于H值，我们计算起点周围每一个方块据红色（终点）的曼哈顿距离，把地图标上 x,y轴，拿到每一个方块的坐标再和终点计算一下就能得到相应的H值。
    
G值和H值都确定了，相加即可得到F值。

好，所有的东西都准备好了，现在可以确定了下一步走到那个方块了。


![image](http://pic002.cnblogs.com/images/2011/70278/2011052614123932.jpg)

之前的三步中，我们已经把起点周围的方块加入到开启列表中了，现在找出开启列表中F值最小的方块，也就是绿色方块右侧的那块，标记为C（F值为40），然后对C进行以下处理：
    
    4.  把它从 "开启列表" 中删除, 并放到 "关闭列表" 中.
    
    5.  检查它所有相邻并且可以到达 (障碍物和 "关闭列表" 的方格都不考虑) 的方格. 如果这些方格还不在 "开启列表" 里的话, 将它们加入 "开启列表", 计算这些方格的 G, H 和 F 值各是多少, 并设置它们的 "父方格" 为 C（这里要注意一下，G值是表示从当前点移动到指定点的代价，所以这一步新加入开启列表的点，要以C点的G值为基准再加相应的代价）.

    6.  如果某个相邻方格 D 已经在 "开启列表" 里了, 检查如果用新的路径 (就是经过C 的路径) 到达它的话, G值是否会更低一些, 如果新的G值更低, 那就把它的 "父方格" 改为目前选中的方格 C, 然后重新计算它的 F 值和 G 值 (H 值不需要重新计算, 因为对于每个方块, H 值是不变的). 如果新的 G 值比较高, 就说明经过 C 再到达 D 不是一个明智的选择, 因为它需要更远的路, 这时我们什么也不做.



到这一步，说明我们从C走是不明智的，所以我们再从开启列表中拿取F值最小的那一个，这时候发现C上下两个方块的F值是一样的，我们随机选一个，选D，D的右边是墙，那我们是否可以直接走到D右下角那个方块(我们标记为E)？这个时候就需要我们在把方块加入开启的列表的时候设置是否可以“走对角”，本文是按照可以走对角的方式。

所以，我们从D方块走到了E方块，从“开启列表”移除了E方块，并加入“关闭列表”，再从“开启列表”中寻找F值最小的方块，再重复上面的步骤，直到目标点进入了“开启列表”中，表示已经找到。

整个过程，就是获取“开放列表”中F值最小的方块，把当前方块加入关闭列表，再加入该方块周围的八个方块，再获取“开放列表”中F值最小的方块.....

最后，被检查过的方块都会进入“关闭列表”，由于F值最小的方块再不断的加入关闭列表，整个路径的走向，按照F值减小，根据G值选择两个方块间最快的移动方式，不断的靠近终点。

想象下面这个过程，会有助于理解F值到底是怎么影响路径的选择的：

    		{0,0,0,0,0,0,0,0,0,0},
			{0,0,0,0,0,0,0,0,0,0},
			{0,0,0,x,x,x,x,0,0,0},
			{0,A,>,>,>,J,x,0,B,0},
			{0,0,0,x,x,x,x,0,0,0},
			{0,0,0,0,0,0,0,0,0,0},
			{0,0,0,0,0,0,0,0,0,0}

如上图，我们需要从A点走到B点，我们设了一个布袋陷阱（x表示障碍物），我们一起看下路径有没有可能一直往右走，然后出不来了？

最开始，按照F值最小和G值最小，路径一直会往右行走（进入布袋陷阱），直到走到J点，发现周围8个点都在“关闭列表”里面（J左侧的点已经被检查过了，障碍点都会被加入“关闭列表”），这个时候看一下上面的步骤，我们啥也不做，而是直接进入了下次循环，从“开启列表”中拿到F值最小的点，这个时候会拿到A点右上角或者右下角的点，拿到另外一条路径，成功逃出“布袋陷阱”。

最后的路径可能如下（*表示路径）：

{% highlight text  %}

    {0,0,0,0,0,0,0,0,0,0},
    {0,0,0,0,0,0,0,0,0,0},
    {0,0,0,x,x,x,x,0,0,0},
    {0,A,>,>,>,J,x,0,B,0},
    {0,0,*,x,x,x,x,*,0,0},
    {0,0,0,*,*,*,*,0,0,0},
    {0,0,0,0,0,0,0,0,0,0}

{%  endhighlight %}
### A*算法实现

主要代码：

~~~ java
while(this.openedList.size() > 0)
{
	currentPoint = getMinPoint(this.openedList);  // get current point
	
	this.closedList.put(currentPoint.key, currentPoint); 
	this.openedList.remove(currentPoint.key);
	
	// get surrounded points
	surroundPoints = this.traversalSurroundPoints(currentPoint);

	it = surroundPoints.iterator();
	
	while(it.hasNext())
	{
		p = it.next();
		
		if (p.walkable == 1 || this.closedList.containsKey(p.key)) continue;
		
		// add new point to opened list
		if (this.openedList.containsKey(p.key) == false)
		{
			p.parent = currentPoint;
			p.g = 	this.getMoveType(p, currentPoint) == 1 
					?  currentPoint.g + this.normalStepCost 
					: currentPoint.g + this.diagonallyStepCost;
			
			this.openedList.put(p.key, p);
		}
		
	    // check 
		if (this.openedList.containsKey(p.key)) {
			int tempCost = 	this.getMoveType(p, currentPoint) == 1 
							? currentPoint.g + this.normalStepCost 
							: currentPoint.g + this.diagonallyStepCost;
			
			if (p.parent != null)
				tempCost += p.parent.g;
			
			// find a least step
			if (tempCost < p.g) {
				p.g = tempCost;
				p.parent = currentPoint;
			}
		}
		
		// found
		if (this.openedList.containsKey(this.end.key)){
			this.generatePaths(this.openedList.get(this.end.key));
			return this.path;
		}
	}
}
~~~ 

源代码：https://github.com/dongyado/awesome-stuff/tree/master/src/top/shares/funny/astar


学习过程中参考的最多的是create chen的[那篇文章](http://www.cnblogs.com/technology/archive/2011/05/26/2058842.html)，对我了解A*有很大的帮助。 本文的出发点，是把自己的对A* 的理解按照自己的思路写出来，作为总结和备忘。
