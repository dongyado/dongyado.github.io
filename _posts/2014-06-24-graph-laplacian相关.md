---
id: 114
title: Graph Laplacian相关
date: 2014-06-24T17:01:26+00:00
author: nicklhy
layout: post
permalink: '/graph-laplacian%e7%9b%b8%e5%85%b3/'
views:
  - "127"
categories:
  - 计算机
---
研一上学期听数学课时第一次听到Graph Laplacian这个词，当时很不明白，现在看Metric Learning时再一次遇见，一样的不太明白，唉，抄公式做个记录看以后能不能理解吧。

首先是两个非常简单易懂的概念，邻接矩阵(Adjacency Matrix)和Degree Matrix(怎么翻译？)，以无向图为例，假设 $$d_i$$ 代表顶点 $$i$$ 的度(因为无向图，所以不分初度和入度)：

$$A_{i,j} = \left\{ \begin{array}{ll} 1 & \textrm{(i,j)\in E}\\ 0 & \textrm{otherwise} \end{array} \right .$$ 

$$D_{i,j} = \left\{ \begin{array}{ll} d_i & \textrm{i=j}\\ 0 & \textrm{otherwise} \end{array} \right .$$ 

对于邻接矩阵，如果仔细观察，可以发现它非常神奇的代表着某种针对顶点函数(即对每个顶点赋值的函数 $$f(v_i)$$ )的运算符：

$$g=Af; \quad g(i) = \sum_{(i,j) \in E}f(j)$$ 

$$f^TAf=\sum_{(i,j) \in E}f(j)^2$$ 

现在借助这两个矩阵，我们可以得到一个叫做Graph Laplacian的矩阵：

$$L=D-A=\left\{\begin{array}{ll} d_i & \textrm{i=j}\\ -1 & \textrm{(i,j) \in E}\\ 0 & \textrm{otherwise} \end{array} \right .$$ 

这个奇怪的Graph Laplacian矩阵乍一眼看上去并没有什么特殊的地方，但是如果仔细观察一下，可以发现它首先是一个对称的半正定矩阵，此外，它同样代表者某种顶点函数的运算符：

$$(Lf)_i=\sum_{(i,j) \in E}(f(i)-f(j))$$ 

$$f^TLf = \sum_{(i,j) \in E}(f(i)-f(j))^2$$ 

实际上如果扩展到有向图(比如把无向图的每条边转化为两条方向相反的有向边)，定义Incidence Matrix( $$\bigtriangledown \in {\mathbb R}^{\|E\|\times \|V\|}$$ ):

$$\bigtriangledown := \left\{ \begin{array}{ll} -1 & \textrm{v is the initial vertex of edge e}\\ 1 & \textrm{v is the terminal vertex of edge e}\\ 0 & \textrm{v is not in e}\end{array} \right .$$ 

我们还可以得到Graph Laplacian的另外一种形式：

$$L=D-A=\bigtriangledown^T\bigtriangledown$$ 

如果把上面这种普通的0、1邻接图改为边带权重的图模型，则有：

$$(Lf)_i=\sum_{(i,j) \in E}W_{ij}(f(i)-f(j))$$ 

$$f^TLf = \frac{1}{2}\sum_{(i,j) \in E}W_{ij}(f(i)-f(j))^2$$ 

对于这个Graph Laplacian，除了用作运算符，还可以用来分析图的一些性质，比如**Fiedler theory**：

假设L的n个特征值和对应的特征向量分别为 $$Lv_i=\lambda v_i, \qquad v_i \neq 0, \qquad i=0,1,\cdots,n-1$$ ，满足 $$0=\lambda_0 \le \lambda_1 \le \cdots \le \lambda_{n-1}$$ ，并且对于第二小的特征向量 $$v_1$$ ，定义：

$$\begin{array}{ll} N_- &= \{i\quad :\quad v_1(i)<0\}\\N_+ &= \{i\quad :\quad v_1(i)>0\}\\N_0 &= \{i\quad :\quad v_1(i)=0\}\end{array}$$ 

则有：

<span style="color: #000000;">(1) $$\#\{i|\lambda_i=0\}=\#\{connected\quad components\quad of\quad G\}$$ ；</span>

<span style="color: #000000;">(2)如果G是连通图，那么 $$N_-$$ 和 $$N_+$$ 都是连通的，如果 $$N_0\neq 0$$ 那么 $$N_-\cup N_0$$ 和 $$N_+ \cup N_0$$ 可能不连通。</span>

对于第一点，我们可以认为Graph Laplacian的0特征值重数等于原图连通分量的个数，例如，若 $$\lambda_1 = 0$$ ，则原图至少存在两个连通分量，若 $$\lambda_1 \neq 0$$ ，则原图是连通的；而对于第二点，可以认为在 $$N_0$$ 为空时，Graph Laplacian为我们提供了一种把原图划分为两个连通分量的方法。