---
layout: post
title: A*寻路的原理和实现 
date: 2015-08-5 12:33:18.000000000 +08:00
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

### 为什么想了解A*算法

平时很少玩游戏，一年一共都不上十次，玩的最多的可能是老掉牙的CS和求生之路， 都是一些FPS游戏，沉迷于打打杀杀的刺激。一直知道游戏的开发是比较复杂的，也不曾去深究。

前段时间给屋里那位买了个WINDOWS平板，里面的扫雷很是不错，角色和玩法是我第一次见的，所以玩的比较多，其中有个很奇妙的现象，吸引了我，那个扫雷的小人可以从障碍中（可能只有一个出口）走出来到鼠标点击的地方，想到了这其中肯定应用了寻路算法。

请教了一下谷歌娘，发现讨论了最多的就是A*算法，初步看了一下，觉得很有趣，就添加到了TODO list里面，准备花点时间学习一下。

### 什么是A*算法

A*（A-Star)算法是一种静态路网中求解最短路径最有效的直接搜索方法。在包含各种障碍物的地图中，为游戏角色的移动，寻找一条到目标地点最短路径。

### A*算法原理

### A*算法实现

学习过程中参考的最多的是create chen的[那篇文章](http://www.cnblogs.com/technology/archive/2011/05/26/2058842.html)，对我学习A*有很大帮助。 这篇博文的出发点，是把自己的对A* 的理解按照自己的思路写出来，作为总结和备忘。
