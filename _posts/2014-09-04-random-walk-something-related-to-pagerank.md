---
id: 264
title: Random walk, something related to pagerank!
date: 2014-09-04T21:55:29+00:00
author: nicklhy
layout: post
permalink: /random-walk-something-related-to-pagerank/
views:
  - "1310"
categories:
  - 计算机
---
OK，今天的主题是**Random Walk**，即**随机游走**，也许这个名字大家会比较陌生，不过没有关系，它在很多领域都有留下自己的身影，比如搞算法的IT程序狗通常第一个能想到的就是Google当年发家的Pagerank算法，虽然这个东西<span style="font-size: 13px;">听起来很高大上，</span>但我们待会再说，先用一个比较接地气的例子对Random Walk解释一番。

桌游大富翁大家小时候应该都玩过，基本就是一个轮流丢骰子的游戏，从初始点开始，看谁先到终点，这里稍作简化，去除金钱、惩罚等各种增加游戏趣味的元素，假设从任意一个点i，经过一步直接到点j的概率是固定的，那么此时就构成了一个典型的Random Walk问题，下面进行数学抽象：

一共有N个点， $$\{1, 2,\cdots, n\}$$ ，任何一个点i可以到达另外一个点j或者不可以，用概率 $$p_{ij} \in [0,1]$$ 进行描述，如果 $$p_{ij}=0$$ ，表示点i不可能直接到达点j，如果 <span style="font-size: 13px;">$$p_{ij}=1$$ ，表示点i在下一步可能且只可能到达点j，如果 $$1>p_{ij}>0$$ ，则表示点i经过一步到达点j的概率为 $$p_{ij}$$ ，如果某人初始位置为i，求解经过n步之后，可能到达哪些点，及对应概率。</span>

其实这个问题本身非常容易，是个典型的Markov Chain on Graph， $$P\in\mathbb R^{N\times N}$$ 是标准的传播矩阵，其中 $$p_{ij}$$ 是i到j的传播概率，且满足row stochastic，即 $$\sum_{j=1}^Np_{ij} = 1$$ ，假设初始状态为 $$x_0$$ ，那么 $$x_n^T = x_0^TP^n, \forall x_0\ge 0, 1^Tx_0=1$$ 。

举个简单的例子，现在有 $$N=5$$ 个点，在每个点往前、往后、保持不动的概率均相等，那么概率转移矩阵

<span style="font-size: 13px;">


  $$P = \left[ \begin{array}{lcr} \frac{1}{2} & \frac{1}{2} & 0 & 0 & 0\\ \frac{1}{3} & \frac{1}{3} & \frac{1}{3} & 0 & 0\\ 0 & \frac{1}{3} & \frac{1}{3} & \frac{1}{3} & 0\\ 0 & 0 & \frac{1}{3} & \frac{1}{3} & \frac{1}{3}\\ 0 & 0 & 0 & \frac{1}{2}& \frac{1}{2} \end{array} \right]$$
</span>

<span style="font-size: 13px;">初始状态假设我们在点1，即 $$x_0=(1, 0, 0, 0, 0)^T$$ ，那么n=3步之后</span>

<span style="font-size: 13px;">


  $$x_3^T = x_0^TP^3 = (0.347, 0.403, 0.194, 0.056, 0)$$
</span>

<span style="font-size: 13px;">也就是说，3步之后，我们在各个点的概率分别是0.347, 0.403, 0.194, 0.056和0。</span>

<span style="font-size: 13px;">根据这个方法，我们也可以很容易得到从点i到点j至少需要多少步，只需要不断迭代， $$x_{k+1}^T = x_k^TP$$ ，直到 $$x_{k+1}$$ 的第j个分量非0，对应的值为到达点j的概率。</span>

现在问题稍微修改一下，**给定一个概率转移矩阵P，从点i到点j，求平均需要多少步。**

这个问题同样可以在前面迭代的思路上做一些修改进行求解：

<ol style="margin-left: 40px;">
  <li>
    $$num\leftarrow 0, k\leftarrow 0$$ ;
  </li>
  <li>
    $$x_{k+1}^T = x_k^TP$$ ;
  </li>
  <li>
    if $$x_{k+1}(j)!=0$$ : $$num \leftarrow num+(k+1)x_{k+1}(j)$$ and $$x_{k+1}(j)\leftarrow 0$$ ;
  </li>
  <li>
    $$k\leftarrow k+1$$
  </li>
  <li>
    if not convergence: return to step 2; else: return num;
  </li>
</ol>

还以上面的例子为例，现在求解从点1到点5需要花费的平均步数，Python 2.7代码如下（需要安装numpy库）：

<pre class="brush:python;">import numpy as np

N = 5
p = np.matrix([[0.5, 0.5, 0, 0, 0], [1./3, 1./3, 1./3, 0, 0], [0, 1./3, 1./3, 1./3, 0], [0, 0, 1./3, 1./3, 1./3], [0, 0, 0, 1./2, 1./2]])
x = np.matrix([1.0, 0, 0, 0, 0])
num = 0.
count = 1
while x.sum()&gt;0.05:
	x = x*p
	if x[0,N-1]!=0:
		num += count*x[0,N-1]
		x[0,N-1] = 0
	count += 1

print 'The average step is %f' % (num)</pre>

运行结果为“The average step is 21.685215”。

注意，这里迭代过程判断收敛的条件是看x里面各分量的和是否足够小，如果足够小，可以认为这一部分对最后结果起的作用很小，因此结束循环，若要从能量的角度来看，则是初始能量大部分已经到达终点5，虽然还剩余很小一部分依旧在图模型中不断传播，但可以略去。

好的，现在问题来了，**这个判断收敛的条件真的可靠吗，有没有可能有一部分能量永远不会到达终点，然后这里变成了死循环？**非常不幸，答案是可能，比如初始点1和另外一个点3构成一个孤立的点对，它们的转移特性就是只互相转移，那么能量将永远不可能到达终点，上面的收敛判断瞬间坑爹不起作用让你CPU持续100%运转。

那么如何避免这种情况呢，数学上有一个专门的性质对此进行描述，叫做Primitive，定义非常简单：设A为一个概率转移矩阵，如果存在 $$k\in\mathbb Z^+$$ ，使得 $$A_{ij}^k>0, \forall i,j$$ ，那么A是Primitive的。

通俗来说，就是图中的任意两个点i、j，最多通过k步总能从i到达j，因此满足Primitive性质的图，能量不会局限在某个小的连通分量里。

由Primitive性质引出的是另一个很重要的定理，即Perron-Frobenius定理：

**Theorem 1.3**(Nonnegative Matrix, Perron-Frobenius) Assume that $$A\ge 0$$ and A is primitive. Then:

<li style="margin-left: 40px;">
  $$\exists \lambda^*>0, v^*>0,\|v^*\|_2=1, s.t. Av^*=\lambda^*v^*$$ (right eigenvector)\\ $$\exists \omega>0, \|\omega^*\|_2=1, s.t. \omega A=\lambda^*\omega$$ (left <span style="font-size: 13px;">eigenvector</span>)
</li>
<li style="margin-left: 40px;">
  $$\forall$$ other eigenvalues $$\lambda$$ of A, $$|\lambda\|<\lambda^*$$
</li>
<li style="margin-left: 40px;">
  $$v^*$$ is unique
</li>

对于一个概率转移矩阵P而言，由于每一行的和都为1，则 $$P\textbf{1}=\textbf{1}$$ ，这里的 $$\textbf{1}$$ 是所有元素均为1的列向量，因此1及时<span style="font-size: 13px;">Perron-Frobenius定理里的 $$v^*$$ ，P的所有其他特征值均满足 $$\lambda \le 1$$ ，于是还存在一个左特征向量 $$\pi$$ 满足 $$\pi^T P=\pi^T$$ ，且 $$\pi^T\textbf{1} = 1, \pi \ge 0$$ ，好了，这里这个 $$\pi$$ 真是一个有趣的特征向量，因为</span>

<span style="font-size: 13px;">


  $$\lim\limits_{k\to\infty}\pi P^k = \pi$$
</span>

<span style="font-size: 13px;">也就是说，我们得到了一个稳定的状态，物体不论再转移多少步，在各个结点的概率均不再变化！别急，还有更加有趣的一件事，如果任意一个转移矩阵P满足Primitive性质，那么随意确定一个初始状态 $$\pi_0$$ ，不断的Random Walk，最终会收敛到这个稳态 $$\pi$$ ！</span>

<span style="font-size: 13px;">


  $$\lim\limits_{k\to\infty}\pi_0 P^k = \pi$$
</span>

OK，题目里还提到了Google高大上的Pagerank，那么这两者有什么关系？<span style="line-height: 1.6em;">我们把整个互联网络抽象成一个图模型 $$G=(V,E,W)$$ ，其中V为网络的结点（如sina、baidu、google这些网站），E为这些结点之间的边（比如我可以通过baidu的一条广告到达可口可乐公司的主页），W为每条边的权重，定义为</span>

<span style="line-height: 1.6em;">


  $$w_{ij} = \left\{ \begin{array}{lc} {\sharp\{\text{link:} i\rightarrow j\} } & {(i,j)\in E}\\ 0 & \text{otherwise}\end{array}\right. $$
</span>

定义出度向量 $$d_i^o = \sum_{j=1}^Nw_{ij}$$ ，表示从结点i出去的link数量；如果所有结点的出度都非零，那么定义对角矩阵 $$D=diag(d_i)$$ 和一个概率转移矩阵 $$P_1=D^{-1}W$$ ，那么我们就可以计算出上面介绍的稳态概率向量 $$\pi$$ ，并且某个结点的对应概率值越大，说明其在互联网中越重要。

等等，我们似乎忘记了<span style="font-size: 13px;">Perron-Frobenius定理的一个前提，就是转移矩阵必须满足Primitive性质，对此，Google的Pagerank算法使用了一个小trick，令 $$P_\alpha = \alpha P+(1-\alpha)E, E=\frac{1}{n}\textbf{1}\textbf{1}^T$$ ，后者E矩阵表示用户可以从一个结点随机跳转到任意一个其他结点， $$\alpha$$ 就控制着用户是跳转与当前结点有关联的结点还是随机跳转的比例，并且附加的E矩阵保证了 $$P_\alpha$$ 是正定的，这也就意味着稳态概率向量 $$\pi$$ 是唯一的（据说Google当年选的 $$\alpha$$ 是0.85）。</span>

再来一个实例，现有一个中国许多高校在网络中链接状况的数据集，<http://pan.baidu.com/s/1hqiR5u0>，Matlab的mat格式，里面有rank\_cn、univ\_cn、W\_cn三个部分，其中rank\_cn是当年的排名数据，后两个一起定义了这写高校的链接状况，构成一个有向带权图，选取不同的 $$\alpha$$ ，我们可以观察一下Pagerank得出的排名状态有什么改变，Python代码如下：

<pre class="brush:python;" style="font-size: 13px">import numpy as np
from scipy.io import loadmat


data = loadmat(&#39;univ_cn.mat&#39;, squeeze_me=True)
rank_cn = data['rank_cn']
W_cn = data['W_cn']
univ_cn = data['univ_cn']
N = W_cn.shape[0]

print &#39;########################################################&#39;
print &#39;univ_rank&#39;
for x in xrange(10):
    print &#39;%d. %s&#39; % (x+1, univ_cn[x])
print &#39;########################################################&#39;

# alpha = 0.85
d = np.array([np.sum(W_cn[i]) for i in range(W_cn.shape[0])])
idnz = filter(lambda i : d[i]&gt;0, range(N))
# D = np.diag(d[idnz])
# W = W_cn[idnz]
p = np.linalg.inv(np.diag(d[idnz])).dot(W_cn[idnz][:,idnz])
P_1 = np.zeros((N, N), dtype=np.float)
# P_1[idnz][:,idnz] = p
for i in range(len(idnz)):
	for j in range(len(idnz)):
		P_1[idnz[i],idnz[j]] = p[i,j]

for alpha in [0.2, 0.4, 0.6, 0.85, 0.95]:
	P_alpha =  alpha*P_1+(1-alpha)*np.ones((N, N), dtype=np.float)/N
	
	(evals, evecs) = np.linalg.eig(P_alpha.T)
	p_eigen_value = evals[0].real
	p_eigen_vector = evecs[:,0].real
	score_page = p_eigen_vector/sum(p_eigen_vector)
	page_rank_id = (-1*score_page).argsort()
	page_rank = univ_cn[page_rank_id]
	
	rho = (page_rank_id-page_rank_id.mean()).dot(rank_cn-rank_cn.mean())/np.sqrt(((page_rank_id-page_rank_id.mean())**2).sum()*((rank_cn-rank_cn.mean())**2).sum())
	print &#39;########################################################&#39;
	print &#39;page_rank(alpha=%f, rho=%f):&#39; % (alpha, rho)
	for i in xrange(10):
		print str(i+1)+". "+str(univ_cn[page_rank_id[i]])
	print &#39;########################################################&#39;
</pre>

<p style="font-size: 13px">
  这里首先输出当年的官方高校排名，然后分别计算了 $$\alpha=\{0.2, 0.4, 0.6, 0.85, 0.95\}$$ 情况下，Pagerank得出的各个高校的排名情况，结果如下，可以看出，各个排名的确存在一些差异，当然，排名仅供娱乐，不必当真（Pagerank的排名P大为毛总在T大后边。。。）。


<hr style="font-size: 13px" />

<p style="font-size: 13px">
  ########################################################<br /> univ_rank<br /> 1. pku.edu.cn<br /> 2. tsinghua.edu.cn<br /> 3. fudan.edu.cn<br /> 4. nju.edu.cn<br /> 5. zju.edu.cn<br /> 6. ustc.edu.cn<br /> 7. sjtu.edu.cn<br /> 8. buaa.edu.cn<br /> 9. nankai.edu.cn<br /> 10. tju.edu.cn<br /> ########################################################<br /> ########################################################<br /> page_rank(alpha=0.200000, rho=0.617069):<br /> 1. tsinghua.edu.cn<br /> 2. pku.edu.cn<br /> 3. uestc.edu.cn<br /> 4. nju.edu.cn<br /> 5. sjtu.edu.cn<br /> 6. zsu.edu.cn<br /> 7. scut.edu.cn<br /> 8. fudan.edu.cn<br /> 9. seu.edu.cn<br /> 10. dlut.edu.cn<br /> ########################################################<br /> ########################################################<br /> page_rank(alpha=0.400000, rho=0.622861):<br /> 1. tsinghua.edu.cn<br /> 2. pku.edu.cn<br /> 3. sjtu.edu.cn<br /> 4. nju.edu.cn<br /> 5. uestc.edu.cn<br /> 6. scut.edu.cn<br /> 7. zsu.edu.cn<br /> 8. fudan.edu.cn<br /> 9. dlut.edu.cn<br /> 10. seu.edu.cn<br /> ########################################################<br /> ########################################################<br /> page_rank(alpha=0.600000, rho=0.640495):<br /> 1. tsinghua.edu.cn<br /> 2. pku.edu.cn<br /> 3. sjtu.edu.cn<br /> 4. nju.edu.cn<br /> 5. uestc.edu.cn<br /> 6. scut.edu.cn<br /> 7. zsu.edu.cn<br /> 8. dlut.edu.cn<br /> 9. fudan.edu.cn<br /> 10. seu.edu.cn<br /> ########################################################<br /> ########################################################<br /> page_rank(alpha=0.850000, rho=0.646343):<br /> 1. tsinghua.edu.cn<br /> 2. pku.edu.cn<br /> 3. sjtu.edu.cn<br /> 4. nju.edu.cn<br /> 5. uestc.edu.cn<br /> 6. scut.edu.cn<br /> 7. zsu.edu.cn<br /> 8. dlut.edu.cn<br /> 9. fudan.edu.cn<br /> 10. seu.edu.cn<br /> ########################################################<br /> ########################################################<br /> page_rank(alpha=0.950000, rho=0.648230):<br /> 1. tsinghua.edu.cn<br /> 2. pku.edu.cn<br /> 3. sjtu.edu.cn<br /> 4. nju.edu.cn<br /> 5. uestc.edu.cn<br /> 6. scut.edu.cn<br /> 7. zsu.edu.cn<br /> 8. dlut.edu.cn<br /> 9. fudan.edu.cn<br /> 10. seu.edu.cn<br /> ########################################################


<p style="font-size: 13px">
  其实除去这个简易版的Pagerank算法，Random Walk还衍生出了其他几种ranking的方法，比如hits authority、hub ranking等，通常在谈到Random Walk、Pagerank时也会提到，不过今天主题不在此，有兴趣的同学可以自己查询下相关资料。
