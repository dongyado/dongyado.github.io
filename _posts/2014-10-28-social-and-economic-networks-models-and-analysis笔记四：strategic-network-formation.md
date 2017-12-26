---
id: 402
title: 'Social and Economic Networks: Models and Analysis笔记四：Strategic Network Formation'
date: 2014-10-28T17:31:10+00:00
author: nicklhy
layout: post
guid: http://closure11.com/?p=402
permalink: '/social-and-economic-networks-models-and-analysis%e7%ac%94%e8%ae%b0%e5%9b%9b%ef%bc%9astrategic-network-formation/'
views:
  - "93"
categories:
  - 计算机
---
之前几节课涉及到的网络大多属于概率随机网络，即结点之间是否存在边是服从一定概率分布的，而这节课即使从名字上看也能发现一些不同，Strategic Network Formation既然包含了Stragetic这个词，那么我们的网络中所有边的构成显然不能在听天由命丢骰子决定了，具体如下。 

## 1.An Economic Analysis
  


首先是一个经济学中的例子，引入变量 $$u_i(g)$$ 为"payoff to i if the network is g"，即在g网络中（无向图），结点i可以获得的收益，那么我们应该如何定义这个变量的计算公式呢？Slides里给了一个很简洁的方式： 

<ul style="margin-left: 40px;">
  <li>
    $$0<\delta<1$$ 为任意两个结点之间如果存在边时这两个结点可以获得的benefit
  </li>
  <li>
    $$0<c_{ij}$$ 是维护任意两个结点之间连接的花费（这里的连接不一定是两点直接相连）
  </li>
  <li>
    $$l(i, j)$$ 是结点i,j之间的最短距离
  </li>
  <li>
    定义 $$u_i(g) = \sum_j\delta^{l(i,j)}-\sum_{j\text{ in }N_i(g)}c_{ij}$$
  </li>
</ul>

注意，在这个公式定义中，一个结点的收益需要考虑连接的收益和开销两方面，前者包括建立连接的数量和每条连接的距离（每条连接随着距离增大，收益变小），后者只考虑i结点与其邻居构成边的数量，距离大于1的连接不再考虑，举个例子，下图显示了根据这个公式计算出的各结点的收益值：[<img alt="1" class="aligncenter size-medium wp-image-412" height="80" src="/images/post/2014/10/12-300x80.png" width="300" srcset="/images/post/2014/10/12-300x80.png 300w, /images/post/2014/10/12-1024x274.png 1024w, /images/post/2014/10/12-690x185.png 690w, /images/post/2014/10/12-980x262.png 980w, /images/post/2014/10/12.png 1409w" sizes="(max-width: 300px) 100vw, 300px" />](/images/post/2014/10/12.png) 

这种网络模型很符合现实中许多不同网络的特性，任何两个人之间要维护比较好的友情/工作/爱情关系都需要付出一些东西（时间、精力等），但同时又能从中得到很多收获，并且通常来说，具有直接联系的两个人/公司，活动会比较密切，得到的收获/收益也会越多，而间接联系的两个人，随着路径长度边长，得到的收益也会逐渐衰减，对于维持两个人/公司的连接，一般来说也只需要维护具有直接交互关系的那些结点，不需要考虑间接相连的。 

## 2.Pairwise Stability and&nbsp;Efficiency
  


既然定义了网络中各个结点的payoff，那么现在我们可以开始考虑评价一个网络的好与坏了。想象人类的关系网络，比如某人刚刚成立了一个创业公司，假设所有员工彼此都不太认识，那么随着时间的推移，这些员工之间会逐渐建立一些连接，比如技术人员可能在小组内部的活动比较频繁，与市场部、财务部的员工不太熟悉，而产品部的员工因为要涉及到各个方面，可能和财务部、市场部、技术部的员工都很熟悉，那么最终这个公司的人际关系网络可能在大尺度上会达到一个稳态，下面我们来模拟这种行为，假设两个结点之间建立一条边需要两个结点同时同意，如果整个网络中任意一个结点改变他和其他结点的announcement状态均不可能使收益变大，那么我们称这个网络达到了一种平衡状态(Nash equilibrium)。 

[<img alt="1" class="aligncenter size-medium wp-image-422" height="153" src="/images/post/2014/10/13-300x153.png" width="300" srcset="/images/post/2014/10/13-300x153.png 300w, /images/post/2014/10/13.png 596w" sizes="(max-width: 300px) 100vw, 300px" />](/images/post/2014/10/13.png)上图中，两个点neither&nbsp;announce或者both&nbsp;announce两种情况都属于<span style="line-height: 20.7999992370605px;">Nash equilibrium，如果一个announce一个不announce（即一个想建立连接，一个不想），则不属于。</span> 

除此之外，定义一种叫做Pairwise Stability的平衡状态，如果图g同时满足下面两个条件： 

<li style="margin-left: 40px;">
  任何一个结点都不能通过删除包含此结点的边来增大 $$u_i(g)$$
</li>
<li style="margin-left: 40px;">
  没有两个结点<span style="line-height: 20.7999992370605px;">通过增加一条连接它们的边</span>可以同时增大各自的收益
</li>

用公式来表示上面两个条件即为： 


  $$u_i(g)\ge u_i(g-ij), \text{ for ij in g}$$



  $$u_i(g+ij)>u_i(g)\Rightarrow u_j(g+ij)$$
<u_j(g), for="" ij="" in="" not="" p=""></u_j(g),> 

在前一个图中，第一种情况属于<span style="line-height: 20.7999992370605px;">Nash equilibrium，但是不符合Pairwise Stability的条件，因为通过添加一条边之后两个结点的payoff均得到了增加，而第二种情况则符合，处于Pairwise Stability状态。</span> 

考虑了平衡相关的评判方式，现在说一说评判一个图&ldquo;好&rdquo;与&ldquo;坏&rdquo;的方式： 

<ul style="margin-left: 40px;">
  <li>
    <span style="color:#FF0000;"><strong>Pareto efficient</strong></span>：如果网络g不存在一个子网络 $$g'$$ 满足 $$u_i(g)\le u_i(g'), \forall i$$ ；
  </li>
  <li>
    <strong><span style="color:#FF0000;">Efficient</span></strong>：g最大化 $$\sum u_i(g')$$
  </li>
</ul>

注意，Pareto efficient定义的反例是存在一个子网络 $$g'$$ 使得该子网络中每个结点的payoff都比g中对应结点要大，而不是某一个结点，Efficient是指网络中所有结点的总收益最大化，下图非常详细的展示了Pareto efficient、Efficient和Pairwise Stable的区别： 

[<img alt="2" class="aligncenter size-medium wp-image-426" height="164" src="/images/post/2014/10/2-300x164.png" width="300" srcset="/images/post/2014/10/2-300x164.png 300w, /images/post/2014/10/2-1024x560.png 1024w, /images/post/2014/10/2-690x377.png 690w, /images/post/2014/10/2-980x536.png 980w, /images/post/2014/10/2.png 1561w" sizes="(max-width: 300px) 100vw, 300px" />](/images/post/2014/10/2.png) 

## 3.Connections Model
  


现在回到1中所定义的payoff，非常明显，参数 $$\delta$$ 和 $$c$$ 的值影响到了一个网络中各个结点收益的大小： 

<li style="margin-left: 40px;">
  low cost网络： $$c<\delta-\delta^2$$ ，完全图是唯一的满足efficient条件的网络；
</li>
<li style="margin-left: 40px;">
  medium cost网络： $$\delta-\delta^2<c<\delta+(n-2)\delta^2/2$$ ，包含所有结点的星形图是唯一满足efficient性质的网络；
</li>
<li style="margin-left: 40px;">
  high cost网络： $$\delta+(n-2)\delta^2/2<c$$ ，不包含任何边的empty network是唯一满足efficient性质的网络。
</li>

Slides中有对这三条结论的简单证明，有兴趣的话可以看一看，这里就不抄公式了。 

上面三条结论主要是针对efficient进行分析，如果考虑到Pairwise Stable的话则会稍微复杂一些： 

<li style="margin-left: 40px;">
  low cost网络： $$c<\delta-\delta^2$$ ，完全图满足Pairwise Stable条件；
</li>
<li style="margin-left: 40px;">
  medium/low cost网络： $$\delta-\delta^2<c<\delta$$ ，包含所有结点的星形图满足Pairwise Stable性质，但不唯一；
</li>
<li style="margin-left: 40px;">
  medium/high cost网络： $$\delta<c<\delta+(n-2)\delta^2/2$$ ，包含所有结点的星形图不满足<span style="line-height: 20.7999992370605px;">Pairwise Stable性质</span>；
</li>
<li style="margin-left: 40px;">
  high cost网络： $$\delta+(n-2)\delta^2/2<c$$ ，不包含任何边的empty network<span style="line-height: 20.7999992370605px;">满足Pairwise Stable性质</span>。
</li>

## 4.Externalities
  


上面依次介绍了结点的平衡状态和网络的&ldquo;好&rdquo;与&ldquo;坏&rdquo;，那么现在来看一看边的<span style="line-height: 20.7999992370605px;">&ldquo;好&rdquo;与&ldquo;坏&rdquo;：</span> 

<ul style="margin-left: 40px;">
  <li>
    <strong><span style="color:#FF0000;">Positive</span></strong>: $$u_k(g+ij)?\ge u_k(g)$$ if ij not in g for every k&ne;i,j
  </li>
  <li>
    <strong><span style="color:#FF0000;">Negative</span></strong>: $$u_k(g+ij)?\le?u_k(g)$$ if ij not in g for every k&ne;i,j
  </li>
</ul>

直接点来说，上面两个性质的意思就是，如果添加一条边ij不会让任意一个结点的收益降低，则称这条边是Positive的，如果不会让任意一个结点的收益升高，则称这条边是Negative的，因此添加Positive的边总是可以增加网络的总收益，添加Negative的边总是降低网络的总收益。 

## 5.Heterogeneity in&nbsp;Strategic Models
  


这一部分再一次提到了Small World模型，通过上面讲述的payoff定义，可以很容易的解释出小世界模型的几个特点： 

<ul style="margin-left: 40px;">
  <li>
    &nbsp;low costs to local links &ndash; high clustering
  </li>
  <li>
    &nbsp;high value to distant connections &ndash; low diameter
  </li>
  <li>
    &nbsp;high cost of distant connections &ndash; few distant links
  </li>
</ul>

&nbsp;
