---
id: 34
title: Real Analysis笔记，chapter 1（Measure Theory）
date: 2014-06-17T10:56:26+00:00
author: nicklhy
layout: post
guid: http://nicklhy.tk/?p=34
permalink: /real-analysis-chapter-1/
views:
  - "76"
categories:
  - 计算机
---
## 一，基本概念

1.首先，以矩形(rectangle)为例，建立体积(volume)的度量(measure)概念：

“In $$\mathbb R$$ we use intervals, while in $${\mathbb R}^d$$ we take products of intervals.”

也就是说，把矩形 $$R=[a_1, b_1]\times [a_2, b_2] \times \cdots \times [a_d, b_d]$$ 的体积定义为： $$\|R\|=(b_1-a_1)\cdots(b_d-a_d)$$ 

2.对于任意一个一维开集，它总是可以由可数(countable)个不相交(disjoint)<span style="color: #ff0000;">开集</span>求并得到，但是对于任意一个 $${\mathbb R}^d (d>1)$$ 空间里的开集而言，情况有些不同，它可以由可数的不相交<span style="color: #ff0000;">闭集</span>求并得到，如果不考虑这些闭集边界重叠的话。

3.**compact set**: bounded and closed set.

并且，任意一个compact set的开覆盖( $$E \in \cup_\alpha {\mathcal O}_\alpha$$ )，都包含一个有限子覆盖(finite subcovering)， $$E \in \bigcup_{j=1}^N {\mathcal O}_{\alpha_j}$$ 。

4.**limit point, isolated point, interior point, closure, boundary.**

5.对于一个rectangle的集合，如何里面每个rectangle的内点都不相交，则称这个rectangle集合almost disjoint。

6.对于任意一个 $${\mathbb R}^d (d \ge 1)$$ 空间上的开子集 $$\mathcal O$$ ，它总是可以写成可数个almost disjoint closed cubes的并集。

7.**Cantor set**: 闭集，有界，compact，totally disconnected，测度（长度）为0，无穷多元素，每个点都不是孤点(isolated point)，not countable。

8.**外测度(exterior measure)**:

$$m_*(E)=\inf\sum^{\infty}_{j=1}\|Q_j\|$$ , with $$E \in \bigcup_{j=1}^\infty {\mathcal Q}_j$$ 

where $$Q_j (j=1,2, \cdots)$$ are closed cubes.

exterior measure与6中定义的形式正好相反，对于任意一个开（闭）集，我们可以从它的开覆盖入手，寻求测量它的方法，也就是找到该集合测度最小的一个开覆盖，且这个最小开覆盖由一系列close cubes构成。

对于exterior measure而言，主要有以下五条性质：

(1)Monotonicity: $$E_1 \in E_2 \Rightarrow m_*(E_1) \le m_*(E_2)$$ .

(2)Countable sub-additivity: $$E=\bigcup_{j=1}^\infty E_j \Rightarrow m_*(E) \le \sum_{j=1}^\infty m_*(E_j)$$ .

(3) $$E \in {\mathbb R}^d \Rightarrow m_*(E)=\inf m_*({\mathcal O})$$ , where $$E \in {\mathcal O}$$ .

(4) $$E=E_1 \cup E_2$$ and $$d(E_1,E_2)>0 \Rightarrow m_*(E)=m_*(E_1)+m_*(E_2)$$ .

(5) $$E=\bigcup_{j=1}^\infty Q_j \Rightarrow m_*(E)=\sum_{j=1}^\infty \|Q_j\|$$ , where $$Q_j$$ are countable almost disjoint cubes.

9.**勒贝格可测(Lebesgue measurable)**：满足对任意一个 $$\epsilon$$ ，存在一个开集 $$\mathcal O$$ 满足 $$E\in {\mathcal O}$$ 和 $$m_*({\mathcal O}-E) \le \epsilon$$ 。