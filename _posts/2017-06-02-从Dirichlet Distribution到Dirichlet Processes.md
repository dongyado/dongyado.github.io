---
id: 1034
title: 从Dirichlet Distribution到Dirichlet Processes
date: 2017-06-02T14:13:15+00:00
author: nicklhy
layout: post
guid: http://closure11.com/?p=1034
permalink: '/%e4%bb%8edirichlet-distribution%e5%88%b0dirichlet-processes/'
views:
  - "36"
dsq_thread_id:
  - "5903428836"
categories:
  - 计算机
---
在过去的几年时间里时常会在书上看到Dirichlet Processes及其相关的一些概念，不过无奈每次都被里面的数学公式吓跑然后放弃，因此一直没有真正的深入学习理解过，最近在看一个聚类算法综述时再次遇到，总算下定决心然后强行撸了几篇tutorial，再结合网上少量的一些中文解释，终于得到了一些粗浅的理解，考虑到自己实在有些健忘，决定在此做些记录。 

## Dirichlet Distribution
  


首先大部分听过Dirichlet Distribution这个名字的人大概都知道它是一个多元概率分布，不过如果你仔细看看维基百科上对它的定义，可能还会发现一个细节就是它不像常见的高斯分布之类能覆盖整个$$R^K$$空间，而是定义在一个K-1维[simplex](https://en.wikipedia.org/wiki/Simplex)上的，那么什么是simplex呢？很简单，你可以认为是三角形在高维空间上的一个扩展，也就是在K-1维空间中由任意K个affinely independent的点 $$u_1, u_2, \cdots, u_K\in R^K$$ 构成的[convex hull](https://en.wikipedia.org/wiki/Convex_hull)，具体可以用以下公式定义 

$$C=\{\pi_1u_1+\cdots\pi_Ku_K|\sum_{k=1}^{K}\pi_k=1\textit{ and }\pi_k\ge 0 \textit{ for all k}\}$$

下图是一个典型的三维simplex： 

<a href="http://closure11.com/wp-content/uploads/2017/06/Tetrahedron.png" rel="" style="" target="" title=""><img alt="" class="aligncenter size-full wp-image-1044" height="240" src="http://closure11.com/wp-content/uploads/2017/06/Tetrahedron.png" style="" title="" width="240" srcset="http://closure11.com/wp-content/uploads/2017/06/Tetrahedron.png 1000w, http://closure11.com/wp-content/uploads/2017/06/Tetrahedron-150x150.png 150w, http://closure11.com/wp-content/uploads/2017/06/Tetrahedron-300x300.png 300w, http://closure11.com/wp-content/uploads/2017/06/Tetrahedron-768x768.png 768w, http://closure11.com/wp-content/uploads/2017/06/Tetrahedron-690x690.png 690w, http://closure11.com/wp-content/uploads/2017/06/Tetrahedron-980x980.png 980w" sizes="(max-width: 240px) 100vw, 240px" /></a> 

现在回到Dirichlet Distribution，它的概率密度公式是 

$$f(\pi_1, \pi_2, \cdots, \pi_K|\alpha_1, \alpha_2, \cdots, \alpha_K) = \frac{\Gamma(\sum_k\alpha_k)}{\prod_k\Gamma(\alpha_k)}\prod_{k=1}^K\pi_k^{\alpha_k-1}$$

满足该分布的一组K维随机变量可以表示为 

$$(\pi_1, \pi_2, \cdots, \pi_K)\sim \textit{Dirichlet}(\alpha_1, \cdots, \alpha_K)$$

注意，由于这里的 $$\pi_1, \cdots, \pi_K$$ 是K-1维空间中一个simplex上的点，所以 $$\sum_{k=1}^K\pi_k=1$$ 且 $$\pi_k\ge 0, \forall k$$ ，这是一个非常重要的性质，因为它使得**我们可以把Dirichlet分布的定义域本身当成一个K维离散概率分布（discrete distributions），也就是说Dirichlet分布是一个K维离散分布的分布（Dirichlet distribution is a probability distribution over pmfs）！** 

好吧，必须承认上面这段描述对于第一次接触到这个概念的人来说可能还是会有些抽象，不过这里我们可以用个更加形象的例子来进行说明：对于某一个特定的六面骰子，它被丢出1-6的概率满足一个固定的离散概率分布，例如这个骰子是个均匀的骰子，那这个离散概率分布应该是 $$p(X=i)=\frac{1}{6}, \forall 1\le i\le 6$$ ，如果这个骰子不是均匀的，那可能变成 $$(1/6, 1/6, 1/2, 1/12, 1/12)$$ 或者其他任意满足 $$\sum_ip(X=i)=1$$ 和 $$p(X=i)>=0, \forall 1\le i\le 6$$ 这两个条件的分布，现在假设某个善于出千的赌王收集了很多很多（趋于无限）各种各样的六面骰子然后装在了一个袋子里，并且每次随机拿出一个概率分布为 $$(\pi_1, \pi_2, \cdots, \pi_6)$$ 的骰子的概率满足上面Dirichlet分布的概率密度公式，那么&ldquo;每次随机拿出的一个骰子&rdquo;就可以被称为服从Dirichlet分布。 

为了方便，我们有时把 $$\textit{Dirichlet}(\alpha_1, \cdots, \alpha_K)$$ 简写成 $$\textit{Dir}(\alpha_1, \cdots, \alpha_K)$$ ，对于 $$\alpha_1=\alpha_2=\cdots=\alpha_K$$ 的特殊的情况，可以简写为 $$\textit{Dir}(\alpha)$$ 。 

由上面的概率公式可以推导出Dirichlet分布的一些有趣性质，比如 

$$(\pi_1, \pi_2, \cdots, \pi_K)\sim \textit{Dirichlet}(\alpha_1, \cdots, \alpha_K)\\ \Rightarrow(\pi_1+\pi_2, \cdots, \pi_K)\sim \textit{Dirichlet}(\alpha_1+\alpha_2, \alpha_3, \cdots, \alpha_K)$$

其它数学性质感兴趣的读者可以查看维基百科或者参考文献中所列的文档，这里不再赘述。 

不过等等，似乎还有一个遗留问题，如果上面这个赌王收集的不是六面骰子，而是各种随机整数生成器呢，即 $$K\rightarrow \infty$$ ？不要着急，这其实是一个无限维Dirichlet分布的问题，并且由于在这种情况下 $$\sum_ip(X=k)=1$$ 和 $$p(X=k)>=0, \forall k\in A$$ 这两个条件依然成立，我们可以通过对原无限空间进行任意划分，例如把A分成 $$\{A_1, A_2, \cdots\}$$ 几个子空间，并令 $$\delta_{k}(A_j)=1 \text{ if } k\in A_j$$ 且 $$\delta_{k}(A_j)=0\text{ otherwise}$$ ，那么有 

$$P(A_j)=\sum^{\infty}_{k=1}p_k\delta_{k}(A_j)$$

由于Dirichlet分布满足上面那条非常好的数学性质，经过我们重新划分的 $$\{A_1, A_2, \cdots\}$$ 依然服从Dirichlet分布，只不过参数有些变化，从 $$\textit{Dirichlet}(\alpha_1, \cdots, \alpha_K)$$ 变成了 $$\textit{Dirichlet}(\alpha(A_1), \alpha(A_2)), \cdots$$ ，其中 

$$\alpha(A_j)=\sum_k^{\infty}\delta_{k}(A_j)\alpha_k$$

## Dirichlet Processes
  


### 数学定义
  


好了，那么到底什么是Dirichlet Process呢？不说废话，先给个数学定义 

  * DP有两个参数： 
      * Base distribution：H，并且是DP过程的均值； 
      * Strength parameter： $$\alpha$$ （一个实数）； 

  * $$G\sim DP$$ 是从DP过程中采样出的一个 $$\mathcal{X}$$ 空间上的概率分布，注意，G本身也是一个概率分布； 

  * 如果对于概率空间 $$\mathcal{X}$$ 的任意满足 $$A_1\cup \cdots \cup A_K = \mathcal{X}$$ 且 $$A_i\cap A_j=\phi, \forall 1 \le i, j \le K, i\ne j$$ 的划分都有$$(G(A_1), \cdots, G(A_K)) \sim \text{Dirichlet}(\alpha H(A_1), \cdots, \alpha H(A_K))$$ ，那么我们说 

    $$G\sim \text{DP}(\alpha, H)$$ 

看完这个定义是否已经懵逼?？第一次看到DP定义的时候反正我心里简直千万匹草泥马奔过，就想抓着作者问这特么是个啥，汗。。。后来反复看了几遍再结合之前Dirichlet分布的定义，总算看出了一些眉目： 

  * DP其实就是从某个概率分布的分布中采样出一个概率分布G，而这个G需要满足一些条件； 
  * &ldquo;H是DP过程均值&rdquo;这句话的意思是，如果把G的所有可能按概率全部积分一遍，得到的就是H； 
  * 为什么要把 $$\mathcal{X}$$ 划分成K个子空间？这是因为G的分布（也就是DP这个分布的分布）没法以一个closed form写出来，但如果我们把 $$\mathcal{X}$$ 这个空间分成了K个子空间，那么G就变成了一个K维离散分布，这时我们是可以给出对应具体的概率分布的； 
  * 为什么要强调&ldquo;**任意**满足xxx条件对 $$\mathcal{X}$$ 的划分&rdquo;？结合前一节我们讨论过的 $$K\rightarrow \infty$$ 的情况，事实上如果 $$(G(A_1), \cdots, G(A_K))$$ 满足Dirichlet分布，那么对 $$A_k$$ 进行任意组合得到的新子空间划分 $$B_1, \cdots, B_{K_2}$$ 依然满足Dirichlet分布，只不过参数有些变化； 
  * "H是DP过程的均值"是说 $$E(G(A))=H(A)$$ ，除了G的均值，事实上它的variance还满足 $$\text{Var}(G(A))=\frac{H(A)(1-H(A))}{1+\alpha}$$ （具体推导过程可以去查阅相关的论文）； 

再等等，看完这个定义和对应的解释是不是觉得很眼熟？这不就是上一节对Dirichlet分布的解释吗？没错，其实上一节那个赌王选骰子的例子以及一切关于&ldquo;从一个概率分布的分布中采样出一个概率分布&rdquo;的相关内容其实都已经算是一种informal的DP概念介绍了，Dirichlet Process之于Dirichlet Distribution就好像Gaussian Process之于Gaussian Distribution，XXX Distribution本身只是一个概率分布，和如何采样是两个完全独立的概念，XXX Process则是对&ldquo;满足XXX概率分布的随机变量进行采样&rdquo;这一过程的描述，算是在XXX分布更高层上的一个概念。 

Gaussian Process和Dirichlet Process应用的intuition都一样，即我们的各种数据可以认为是来自某一确定随机过程所产生的一个概率分布所产生的一组观测值，换句话来说，就是先通过这个随机过程产生一个概率分布，再从这个概率分布中抽样得到一组观测值（真是有点绕?）；另外由于这不是一个能被某几个参数唯一确定的模型，Dirichlet Process和Gaussian Process都属于non-parametric方法（具体可以参考[这里](https://www.zhihu.com/question/54354940)）。 

### 如何通过观测到的数据计算原始分布？
  


现在继续Dirichlet Process相关的内容，假设我们从 $$G\sim \text{DP}(\alpha, H)$$ 采样得到随机变量 $$\theta \sim G$$ ，然后在 $$\theta$$ 这一离散分布中我们又要采样得到随机变量Z，对我们来说显然能够直接被观测到的只有Z的一系列样本 $$z_1, z_2, \cdots$$ ，而目标确实估计 

  $$p(\theta)=\int p(\theta|G)p(G)dG$$

和 

  $$p(G|\theta)=\frac{p(\theta |G)p(G)}{p(\theta)}$$

另外由于 $$\theta=(\pi_1, \cdots, \pi_K)\sim \text{Dirichlet}(\alpha_1, \cdots, \alpha_K)$$ 且 $$Z|(\pi_1, \cdots, \pi_K)\sim \text{Discrete}(\pi_1, \cdots, \pi_K)$$ ，利用贝叶斯公式经过一定推导可以得到 

  $$Z\sim \text{Discrete}(\frac{\alpha_1}{\sum^K_{k=1}\alpha_k}, \cdots, \frac{\alpha_K}{\sum^K_{k=1}\alpha_k})$$

  $$\theta|Z=(\pi_1, \cdots, \pi_K)|Z \sim \text{Dirichlet}(\alpha_1+\delta_1(Z), \cdots, \alpha_K+\delta_K(Z))$$

这里 $$\delta_k(Z)=1$$ 如果 $$Z=k$$ ，否则等于0。 

### Dirichlet Process为什么是个随机过程？
  


前面讲了一堆DP的定义，不过怎么看起来完全就是在一个概率分布上采样的过程？DP作为一种随机过程，它难道不该是在时间轴上不断随机生成数据样本 $$\theta_1, \theta_2, \cdots$$ 么？其实这个想法没错，事实上最初的DP就是这么定义的 

  * Input: H, $$\alpha$$ . 
  * Steps: 
      1. $$\theta_1 \sim H$$ 
      2. For $$n>1$$ ,
          $$\theta_n(\theta)|\theta_1, \theta_2, \cdots, \theta_{n-1} \sim \frac{\alpha H}{\alpha+n-1}+ \frac{\sum_{i=1}^{n-1}\delta_{\theta_i}(\theta)}{\alpha+n-1}$$

这个DP的实现过程叫做"Blackwell-MacQueen Urn Scheme"，在维基百科上还有一个类似的实现过程，但是差别在于采样 $$\theta_n$$ 时把上面公式的两项拆开了，即以概率 $$\frac{\alpha}{\alpha+n-1}$$ 从H采样，以概率 $$\frac{\sum_{i=1}^{n-1}\delta_{\theta_i}(\theta)}{\alpha+n-1}$$ 设置成过去的某个 $$\theta_i$$ 的值，这种方法叫做"Polya Urn"，这两种方法的差异个人认为是在不同应用场景下对采样有不一样的需求导致的，不过核心思想完全一致；除了这里提到的两种DP构造方式，其实还有很多其他的做法，例如比较出名的"Chinese Restaurant Process(CRP)"、"Stick-breaking Construction"。 

不过目前看起来这种在时间轴上不断生成的 $$\theta_1, \theta_2, \cdots$$ 似乎又和之前所说的 $$G$$ 不太一样了，毕竟之前从 $$G$$ 采样 $$\theta_1, \theta_2, \cdots$$ 时它们完全是iid的（也就是互相独立且服从相同的分布），为什么现在变成 $$\theta_n$$ 依赖 $$\theta_1, \cdots, \theta_{n-1}$$ 了？另外，之前花了这么多时间去尝试理解的Dirichlet分布怎么在这里直接消失了？这两个DP的描述到底有啥关系？ 

现在来解答上面的问题：

首先这两个DP描述说的都是DP这一个概念（好吧，这完全是一句废话）；把联合分布 $$p(\theta_1, \cdots, \theta_{n}\vert H, \alpha)$$ 推导出来（de Finettis theorem），可以发现在形式上它与 $$\theta_i$$ 的顺序无关，而只与H和 $$\alpha$$ 有关，也就是和G有关，因此我们说 $$\theta_1, \theta_2, \cdots$$ 是 $$\text{iid}\sim G$$ ， $$\theta_1, \theta_2, \cdots$$ 是**exchangeable**的；事实上，上面 $$\text{DP}(\alpha, H)$$ 的定义是从这一节提到的各种DP实现过程构造出来的。 

总结一下，DP首先是从类似"Blackwell-MacQueen Urn Scheme"、"Chinese Restaurant Process"等过程实现的，但是由于其满足的一些特殊条件，使得生成的概率分布序列 $$\theta_1, \theta_2, \cdots$$ 互相独立，不再与时间顺序有关，只受G所影响，于是有了前面最开始提到的那个定义。 

（关于从随机过程的来解释DP还可以参考知乎这个[页面](https://www.zhihu.com/question/22717561)中豆豆页的回答。） 

&nbsp; 

## 结束语
  


上面虽然说了挺多Dirichlet分布和DP的内容，但大多还是着眼于它们的理论解释，在实际问题中，DP可以用来做概率密度估计、聚类等，虽然在各大互联网公司的程序员群体中似乎不是特别出名，但仍有适合的应用场景，本篇先在此收尾，未来看情况考虑再写几篇专门介绍实际应用的文章。 

&nbsp; 

## 参考文献
  


1. [Dirichlet Processes: Tutorial and Practical Course](https://link.zhihu.com/?target=http%3A//www.stats.ox.ac.uk/%7Eteh/teaching/npbayes/mlss2007.pdf);
