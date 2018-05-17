---
id: 57
title: Structural SVM原理分析
date: 2014-06-19T23:51:20+00:00
author: nicklhy
layout: post
permalink: /structural-svm/
views:
  - "1183"
categories:
  - 计算机
---
对于大多数机器学习中的分类器而言，每个样本由输入 $$x \in {\mathcal X}$$ 、输出 $$y \in {\mathcal Y}$$ 构成，而我们的任务就是寻找一个针对输入到输出的映射，即 $${\mathcal X} \rightarrow {\mathcal Y}$$ ，在传统方法中，通常输入空间属于多维空间 $${\mathcal X} \in {\mathbb R}^d$$ ，而输出等于某个有限大小离散集合&nbsp; $$\{1, 2, 3, \cdots , N\}$$ 中的一个数，虽然用一个数字来表示输出在大多数情况下都适用，但在诸如视觉、自然语言处理等一些特殊领域应用时可能会存在一些局限性，因为输出可能并不能由一个一维数字比较完整的代表，于是structured learning应运而生，在这里，输出y被允许取值为和输入x类似的多维结构，以此提出和实际情况更加符合的模型，这里以Structural SVM为例进行说明，主要参考文献:&nbsp;<span style="color: #ff0000;"><span style="color: #ff0000;"><a href="http://www.cs.cornell.edu/people/tj/publications/tsochantaridis_etal_04a.pdf">Support vector machine learning for interdependent and structured output spaces.&nbsp;Tsochantaridis I, Hofmann T, Joachims T, et al. ICML 2004.</a></span></span> 

假设我们的模型参数为 $$w$$ ，输入输出分别为 $$x \in {\mathcal X}$$ 和 $$y \in {\mathcal Y}$$ ，我们的目标是学习一个用以预测的判别函数f： 

$$\hat{y} = f(x;w) = \arg\max_{y \in {\mathcal Y} }F(x, y; w)$$ 

（当然我们也可以设计一个类似-F这样的函数当做损失函数，然后求其最小值）。 

我们假设F与输入、输出的某种综合特征具有线性关系： 

$$F(x, y; w) = \langle w, \Psi(x, y)\rangle$$ 

具体选取的特征 $$\Psi(x, y)$$ 随实际应用场景而定。 

此外，我们还需要设计一个对预测结果评价的损失函数 $$\Delta: {\mathcal Y} \times {\mathcal Y} \rightarrow {\mathbb R}$$ ，并且如果预测值和真实值越接近，损失应该越小，反之，则越大，于是总的risk可以定义为： 

$$R_P^\Delta = \int_{ {\mathcal X} \times {\mathcal Y} }\Delta(y,f(x))dP(x,y)$$ 

这里P是数据的概率分布，一般对我们而言都是未知的，所以需要用训练样本数据计算的empirical risk $$R_S^\Delta$$ 来代替。 

理想情况下，应该可以找到一个或多个参数w使得在训练样本 $$S=\{(x_i,y_i)\in {\mathcal X}\times {\mathcal Y}: i=1,2,\cdots,n\}$$ 上计算出的empirical risk为0，并且其条件为： 

$$\forall i : \qquad \max_{y \in {\mathcal Y\backslash y_i} }\{\langle w, \Psi(x_i, y)\rangle \} < \langle w, \Psi(x_i, y_i) \rangle$$ 

上式由n个包含max函数的linear inequalities约束条件构成，可以继续展开成 $$n\|{\mathcal Y}\|-n$$ 个普通的式子： 

$$\forall i, \forall y \in {\mathcal Y} \backslash y_i \quad : \quad \langle w,\delta \Psi_i(y)\rangle > 0$$ 

这里 $$\delta \Psi_i(y) = \Psi(x_i, y_i)-\Psi(x_i, y)$$ 。 

对于上面的多个约束条件而言，可能会由多个解，这里可以借助经典SVM的maximum-margin原则，在 $$\|w\|\le 1$$ 的情况下，使得hyper-plane和最近的数据点距离最大，于是可以把原问题转化成一个类似经典SVM的优化问题： 

$${SVM}_0 \quad : \quad \min_w \frac{1}{2}\|w\|^2$$ 

$$\forall i, \forall y \in {\mathcal Y} \backslash y_i \quad : \quad \langle w,\delta \Psi_i(y) \rangle \ge 1$$ 

然后加上软边界得到： 

$${SVM}_1 \quad : \quad \min_w \frac{1}{2}\|w\|^2+\frac{C}{n}\sum_{i=1}^{n}\xi_i,\quad s.t. \quad \forall i, \xi_i\ge 0$$ 

$$\forall i, \forall y \in {\mathcal Y} \backslash y_i \quad : \quad \langle w,\delta \Psi_i(y)\rangle \ge 1-\xi_i$$ 

这里加上的是margin violations $$\xi_i$$ 的和，文章还提到可以换成其二范数和，得到另一种SVM变形；观察从Structural SVM提取出来的数学优化问题，可以看出其目标函数其实与经典SVM类似，都是通过最小化 $$\|w\|^2$$ 来实现maximum-margin思想，然后附加一个软边界regularizer防止异常点的存在使超平面偏离太离谱，所不同的地方主要在于地下的约束条件，经典SVM通过限制样本点分布在超平面两侧并且保持足够的距离，而Structural SVM则是通过限制事先构造的 $$x,y$$ 损失函数大小来约束，这里避免了前者简单粗暴的惩罚方式(比如分错惩罚1，分对惩罚0)，对分类错误程度不同的样本进行程度不同的惩罚，显然会对分类器的准确率有比较好的提高。
