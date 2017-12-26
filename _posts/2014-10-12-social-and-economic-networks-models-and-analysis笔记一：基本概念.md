---
id: 327
title: 'Social and Economic Networks: Models and Analysis笔记一：基本概念'
date: 2014-10-12T16:07:38+00:00
author: nicklhy
layout: post
guid: http://closure11.com/?p=327
permalink: '/social-and-economic-networks-models-and-analysis%e7%ac%94%e8%ae%b0%e4%b8%80/'
views:
  - "115"
categories:
  - 计算机
---
Social and Economic Networks: Models and Analysis这门课是斯坦福大学在Coursera上的一门公开课，恰好这段时间开始开放，加上<span style="line-height: 20.7999992370605px;">评分还不错，</span>于是打算来听听，由于自己对Social Network Analysis不是特别熟悉，所以就从第一节课一些基本概念开始，在这个博客留下一些笔记吧。 

第一节课主要涉及<span style="line-height: 20.7999992370605px;">社会网络</span>分析的一些基本问题和概念，例如为什么Facebook这样的<span style="line-height: 20.7999992370605px;">社会网络</span>里大部分人之间都具有很短的图上距离？社会网络如何影响人类行为？下面是具体内容： 

1.n个结点可以形成多少种不同的网络？ 

考虑所有可能存在的边的数量为 $$n+(n-1)+\cdots+1=\frac{n(n+1)}{2}$$ ，所以可能形成的网络的数量为 $$2^{\frac{n(n+1)}{2} }$$ 。 

2.Network表示方法 

用邻接矩阵G表示，若 $$G_{ij}=1$$ ，表示存在一条从顶点i到j的边；若为无向图，则G为对称矩阵，若为有向图，则不一定；若为带权图，则 $$G_{ij}$$ 表示边 $$i\rightarrow j$$ 的权重，否则，1表示连通，0表示不连通。 

3.图相关概念的基本定义： 

  * Path（路径）:&nbsp;a walk $$(i_1, i_2, \cdots, i_K)$$ with each node $$i_k$$ distinct 
  * Cycle（环路）: a walk where $$i_1=i_K$$ 
  * Geodesic（测地线）: a shortest path between two nodes 
  * Components(连通分量): maximal connected subgraph 
  * Diameter（直径？）:&nbsp;&nbsp;largest geodesic (largest shortest path) 
  * Neighborhood（邻居）: $$N_i(g) = \{ j | G_{ij}=1\}$$ 
  * Degree（度数）: $$d_i = # N_i(g)$$ 

4.Erdos‐Renyi&nbsp;Random Graphs 

这是一个很重要的随机图模型，概念很简单：初始n个点，然后任意两点之间边存在的概率为p，整个图表示为G(n, p)。 

关于这个Erdos-Renyi随机网络，有一些特性： 

  * 如果所有结点的平均度数 $$d(n) \gt (1+\epsilon) log(n) some \epsilon>0$$ ，那么整个网络很可能是连通的； 
  * 如果 $$d(n) \gt (1+\epsilon) log(n) some?\epsilon>0$$ 并且 $$\frac{d(n)}{n}\rightarrow 0$$ ，那么当n足够大时，网络的平均路径长度和Diameter将正比于 $$\frac{log(n)}{log(d)}$$ 。 

5.树 

树也是一种特殊的网络，这里考虑一种特殊的树，其形成方式为： 

<p style="margin-left: 40px;">
  (a)初始一个根<span style="line-height: 20.7999992370605px;">节点</span>


<p style="margin-left: 40px;">
  (b)根结点产生d个<span style="line-height: 20.7999992370605px;">子节点</span>


<p style="margin-left: 40px;">
  (c)b中生成的d个子<span style="line-height: 20.7999992370605px;">节点</span>每个再产生(d-1)个子节点


<p style="margin-left: 40px;">
  ......


[<img alt="1" class="aligncenter size-medium wp-image-346" height="241" src="/images/post/2014/10/1-300x241.jpg" width="300" srcset="/images/post/2014/10/1-300x241.jpg 300w, /images/post/2014/10/1-280x225.jpg 280w, /images/post/2014/10/1.jpg 527w" sizes="(max-width: 300px) 100vw, 300px" />](/images/post/2014/10/1.jpg)这种树在经过l步之后，将包含 $$1+d+d(d-1)+\cdots +d(d-1)^{l-1}=1+\frac{d((d-1)^l-1)}{(d-2)}\approx (d-1)^l$$ 个节点。 

6.Chernoff Bounds 

若X是一个二项分布的随机变量，那么 $$P(E[X]/3\le X \le 3E[X])\gt 1-e^{-E[X]}$$ 。 

应用到ER图中，那么顶点度数比较靠近平均值d的概率 $$P(d/3\le d_i \le 3d)\gt 1-e^{-d}$$ 。 

7.Clustering 

What fraction of my friends are friends of each other? 

$$Cl_i(g)=#\{\text{kj in g}| \text{k,j in } N_i(g)\}/#\{\text{k,j in } N_i(g)\}$$ 

Average Clustering: $$Cl^{Avg}(g)=\sum_iCl_i(g)/n$$
