---
id: 390
title: 'Social and Economic Networks: Models and Analysis笔记三：Random Networks'
date: 2014-10-21T21:44:20+00:00
author: nicklhy
layout: post
guid: http://closure11.com/?p=390
permalink: '/social-and-economic-networks-models-and-analysis%e7%ac%94%e8%ae%b0%e4%b8%89%ef%bc%9arandom-networks/'
views:
  - "75"
categories:
  - 计算机
---
## 1.Growing Random Network
  


这部分其实介绍的是一种Uniformly Growing的随机网络，满足： 

  * 初始m个点，任意两点均有边相连； 
  * 每单位时间产生一个新的点，并且在已存在的点中随机选择m个点，构成m条新的边； 
  * 上一步在已存在的所有点中选择m个点时，为等概率随机选择，即每个点被选中的概率为m/t（感觉slides里的意思，t时刻已存在点的个数为t，产生了第t+1个点，但这样初始时刻就不是t=0而是t=m了）。 

那么对于任意一个点i，在m<i<t时间段内产生，它的度数的期望为： 


  $$m+\frac{m}{i+1}+\frac{m}{i+2}+\cdots+\frac{m}{t}$$


上式近似等于 


  $$\text{approx} = m(1+\log{(t/i)})$$


那么如果i和t满足： 


  $$\frac{i}{t} > te^{-(d-m)/m}$$


则这些点的度数期望将小于d，这也意味着，所有t个点中，度数小于d的点的比例为 $$F_t(d) = 1-e^{-(d-m)/m}$$ 。 

其实如果把时间变为连续，度数期望也可以从一个简单的微分方程解出来： 


  $$dd_i(t)/dt = m/t \text{ and } d_i(i) = m \Longrightarrow d_i(t) = m+m\log{(t/i)}$$


## 2.Power Law
  


上面所说的Uniformly Growing Random Network有一个很大的约束条件，就是新结点与旧结点形成边的时候是等概率选择，这与实际中很多网络都不符合，比如参考文献的引用网络，一般情况下都是越好越新的文章被引用概率越大，而很多差的文章几乎不会被引用，因此形成的网络就会有很多high degree和low degree的结点，而非我们之前预测的那样，我们称这种情况为fat tails，然后把这种rich get richer的现象总结为一条定律，即Power Law。对比之前的Uniformly Growing Random Network，把&ldquo;等概率选择&rdquo;改为&ldquo;新结点和旧结点形成边的概率正比于该旧结点已存在边的数量&rdquo;即可形成符合Power Law的新网络。 

## 3.Preferential Attachment
  


对于上面所述这种带有偏好的随机网络，我们又称其为Preferential Attachment： 

  * 新结点与已存在的结点构成m条新的边 
  * t时刻整个网络一共有tm条边 
  * Total degree为2tm 
  * 新结点和已存在的某一结点i构成边的概率为 $$d_i(t)/2tm$$ 

抽象一下，即可得到 $$dd_i(t)/dt = md_i(t)/2tm=d_t(t)/2t, d_i(i) = m$$ 两个式子，如果以连续时间作为前提，求解出来即可得到： 


  $$d_i(t) = m(t/i)^{1/2}$$


此时，t时刻所有t个点中度数小于d的比例则为： 


  $$F_t(d) = 1-m^2/d^2$$


概率密度函数 $$f_t(d) = 2m^2/d^3$$ 。 

## 4.Hybrid Model
  


这里就是把上面Uniformly Growing和遵循Power Law的两种模型进行混合， 


  $$dd_i(t)/dt = am/t+(1-a)d_i(t)/2t \text{ and } d_i(i) = m$$


解出来是个有点复杂的式子 


  $$d_i(t) = (m+2am/(1-a))(t-i)^{(1-a)/2}-2am/(1-a)$$


此时如果计算t时刻所有t个结点中度数小于d的结点数量，并用 $$x=2/(1-a)$$ 换元，得到 


  $$i/t = [(m+xam)/(d+xam)]^x$$



  $$F(d) = 1-((m+amx)/(d+amx))^x$$


&nbsp; 

&nbsp;