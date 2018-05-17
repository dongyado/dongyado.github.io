---
id: 705
title: RCNN, Fast-RCNN, Faster-RCNN的一些事
date: 2016-03-18T16:13:02+00:00
author: nicklhy
layout: post
permalink: '/rcnn-fast-rcnn-faster-rcnn%e7%9a%84%e4%b8%80%e4%ba%9b%e4%ba%8b/'
views:
  - "18964"
dsq_thread_id:
  - "5904971312"
categories:
  - 计算机
---
rbg大神的深度神经网络检测算法系列RCNN、Fast-RCNN、Faster-RCNN可谓是理论与实践的经典范例，论文创新点足够，在github上开源的代码更是造福广大码农，本文以当前最新Faster-RCNN的python实现（<https://github.com/rbgirshick/py-faster-rcnn>）为准，尝试对rcnn系列算法中的几个关键核心点进行详细的分析： 

  * RCNN -> Fast-RCNN -> Faster-RCNN 
  * 图片区域分析核心：ROI Pooling层 
  * 对象bbox预测：Bounding-box Regression 
  * 用神经网络输出proposal：RPN层 
  * Faster-RCNN训练步骤 

## RCNN -> Fast-RCNN -> Faster-RCNN
  


这里不得不先提的就是为什么会有RCNN这一系列的检测算法，以及为什么它们会被称为深度对象检测的开山之作，我们知道，在CNN火起来之前，对象检测这一问题基本是遵循着&ldquo;设计手工特征(_Hand-crafted feature_)+分类器&rdquo;的思路，而且由于存在着区域搜索的步骤，所以可以认为是计算机用一个小的矩形窗口不断在图像上滑动、缩放，然后用分类器预测当前滑动窗口所在区域是否存在一个感兴趣的对象，自从CNN在CV领域流行起来以后，很多人都开始想，既然CNN的特征比传统手工特征好这么多，那么为什么不用深度神经网络做检测呢？ 

RCNN算法的核心思想就是对每个区域通过CNN提取特征，然后接上一个分类器预测这个区域包含一个感兴趣对象的置信度，也就是说，转换成了一个图像分类问题（类似imagenet），后面接的这个分类器可以是独立训练的svm也可以是简单的softmax分类。在RCNN论文里，作者还提到两个保证检测速度的关键点：1.所有类别的分类器共享相同的特征输入；2.与传统特征相比，深度特征维度一般比较低，比如VGG16里的4096维。 

但是很可惜，即使使用了selective search等预处理步骤来提取潜在的bounding box作为输入，但是RCNN仍会有严重的速度瓶颈，原因也很明显，就是计算机对所有region进行特征提取时会有重复计算，Fast-RCNN正是为了解决这个问题诞生的，作者提出了一个可以看做单层sppnet的网络层，叫做ROI Pooling，这个网络层可以把不同大小的输入映射到一个固定尺度的特征向量，而我们知道，conv、pooling、relu等操作都不需要固定size的输入，因此，在原始图片上执行这些操作后，虽然输入图片size不同导致得到的feature map尺寸也不同，不能直接接到一个全连接层进行分类，但是可以加入这个神奇的ROI Pooling层，对每个region都提取一个固定维度的特征表示，再通过正常的softmax进行类型识别。另外，之前RCNN的处理流程是先提proposal，然后CNN提取特征，之后用SVM分类器，最后再做bbox regression，而在Fast-RCNN中，作者巧妙的把bbox regression放进了神经网络内部，与region分类和并成为了一个multi-task模型，实际实验也证明，这两个任务能够共享卷积特征，并相互促进。Fast-RCNN很重要的一个贡献是成功的让人们看到了Region Proposal+CNN这一框架实时检测的希望，原来多类检测真的可以在保证准确率的同时提升处理速度，也为后来的Faster-RCNN做下了铺垫。 

Fast-RCNN之后的问题已经非常清晰，就是我们能不能把region proposal部分也放到GPU上？rbg大神给的答案当然又是yes，于是有了Faster-RCNN，出现了一个end-to-end的CNN对象检测模型。作者提出，网络中的各个卷积层特征其实可以用来预测类别相关的region proposal，不需要事先执行诸如selective search之类的算法，但是如果简单的在前面增加一个专门提proposal的网络又显得不够elegant，所以最终把region proposal提取和Fast-RCNN部分融合进了一个网络模型，虽然训练阶段仍然要分多步，但是检测阶段非常方便快捷，准确率也与原来的Fast-RCNN相差不多，从此，再也不用担心region proposal提取耗时比实际对象检测还多这种尴尬场景了。 

## ROI Pooling
  


首先需要介绍RCNN系列里的一个核心算法模块，即ROI Pooling。我们知道在ImageNet数据上做图片分类的网络，一般都是先把图片crop、resize到固定的大小（i.e. 224*224），然后输入网络提取特征再进行分类，而对于检测任务这个方法显然并不适合，因为原始图像如果缩小到224这种分辨率，那么感兴趣对象可能都会变的太小无法辨认。RCNN的数据输入和SPPNet有点类似，并不对图片大小限制，而实现这一点的关键所在，就是ROI Pooling网络层，它可以在任意大小的图片feature map上针对输入的每一个ROI区域提取出固定维度的特征表示，保证后续对每个区域的后续分类能够正常进行。 

ROI Pooling的具体实现可以看做是针对ROI区域的普通整个图像feature map的Pooling，只不过因为不是固定尺寸的输入，因此每次的pooling网格大小得手动计算，比如某个ROI区域坐标为 $$(x_1, y_1, x_2, y_2)$$ ，那么输入size为 $$(y_2-y_1)*(x_2-x_1)$$ ，如果pooling的输出size为 $$pooled\_height*pooled\_width$$ ，那么每个网格的size为 $$\frac{y_2-y_1}{pooled\_height}*\frac{x_2-x_1}{pooled\_width}$$ ，具体代码可在roi\_pooling\_layer.cpp中的Forward_cpu
	  
函数里找到，比较简单。 

作者并没有对Backward阶段实现CPU代码，所以只能在roi\_pooling\_layer.cu中查看，即ROIPoolBackward函数，其具体进行的操作可以用论文里的一行公式形容， 


  $$\frac{\partial{L} }{\partial{x} } = \sum_{r\in R} \sum_{y \in r} [\text{y pooled x}] \frac{\partial{L} }{\partial{y} } $$


其中 $$R$$ 表示R个输入ROI区域以及对应的R个输出feature，x和y分别表示输入的feature map和输出的feature，整个公式的意思就是，"During back-propagation, derivatives&nbsp;flow through the RoI pooling layer. The RoI pooling layer's&nbsp;backwards function computes the partial derivative of the&nbsp;loss function with respect to each input variable x by summing&nbsp;over all RoIs that max-pooled x in the forward pass."，另外，由于实际实现是采用的是Max Pooling，因此y pooled x表示&ldquo;x在该网格区域中最大，然后y被assign到x的值&rdquo;，而具体每个网格中哪个点的值最大，也是在Forward过程中就已经记录，存储在了argmax_data变量里。 

## Bounding-box Regression
  


有了ROI Pooling层其实就可以完成最简单粗暴的深度对象检测了，也就是先用selective search等proposal提取算法得到一批box坐标，然后输入网络对每个box包含一个对象进行预测，此时，神经网络依然仅仅是一个图片分类的工具而已，只不过不是整图分类，而是ROI区域的分类，显然大家不会就此满足，那么，能不能把输入的box坐标也放到深度神经网络里然后进行一些优化呢？rbg大神于是又说了"yes"。在Fast-RCNN中，有两个输出层：第一个是针对每个ROI区域的分类概率预测， $$p=(p_0, p_1, \cdots, p_K)$$ ；第二个则是针对每个ROI区域坐标的偏移优化， $$t^k = (t^k_x, t^k_y, t^k_w, t^k_h)$$ ， $$0 \le k \le K$$ 是多类检测的类别序号。这里我们着重介绍第二部分，即坐标偏移优化。 

假设对于类别 $$k^*$$ ，在图片中标注了一个groundtruth坐标： $$t^* = (t^*_x, t^*_y, t^*_w, t^*_h)$$ ，而预测值为 $$t = (t_x, t_y, t_w, t_h)$$ ，二者理论上越接近越好，这里定义损失函数： 


  $$L_{loc}(t, t^*) = \sum_{i \in \{x, y, w, h\} } \text{smooth}_{L_1}(t_i, t^*_i)$$


其中 


  $$\text{smooth}_{L_1}(x) = \left \{ \begin{aligned} &0.5x^2 & |x| \le 1\\ &|x|-0.5 & \text{otherwise}\end{aligned} \right.$$


这里， $$smooth_{L_1}(x)$$ 中的x即为 $$t_i-t^*_i$$ （感觉前一个公式为作者笔误，该写成 $$\text{smooth}_{L_1}(t_i-t^*_i)$$ ），即对应坐标的差距。该函数在 $$(-1, 1)$$ 之间为二次函数，而其他区域为线性函数，作者表示这种形式可以增强模型对异常数据的鲁棒性，整个函数在matplotlib中画出来是这样的 

<a href="/images/post/2016/01/smooth.png" rel="attachment wp-att-740"><img alt="smooth" class="aligncenter size-medium wp-image-740" height="225" src="/images/post/2016/01/smooth-300x225.png" width="300" srcset="/images/post/2016/01/smooth-300x225.png 300w, /images/post/2016/01/smooth-768x576.png 768w, /images/post/2016/01/smooth-690x518.png 690w, /images/post/2016/01/smooth.png 800w" sizes="(max-width: 300px) 100vw, 300px" /></a> 

对应的代码在smooth\_L1\_loss_layer.cu中。 

## RPN层
  


Faster-RCNN最大一点贡献应该算是其把proposal部分从网络外边嵌入了网络里边，从此一个网络模型即可完成end-to-end的检测任务而不需要我们在前面手动先执行一遍proposal的搜索算法。其实如果回过头来看看几年前比较流行的检测算法，比如HOG+SVM和DPM什么的，同样是需要用分类器逐个对一些矩形框里提取出来的特征进行分类，只不过那时是全图设置好stride、scale等参数然后搜索，不像selective search这些算法会去对图像进行内容分析，然后输出一些可疑的矩形候选框。 

某种程度上，RPN也可以算作一个全图搜索的粗检测器，图片在输入网络后，依次经过一些卷积、池化层，然后得到的feature map被手动划分为 $$n\times n$$ 个矩形窗口（论文中n=3），准备后续用来选取proposal，并且此时坐标依然可以映射回原图。需要注意两点问题：1.在到达全连接层之前，卷积层和Pooling层对图片输入大小其实没有size的限制，因此RCNN系列的网络模型其实是不需要实现把图片resize到固定大小的；2.n=3看起来很小，但是要考虑到这是非常高层的feature map，其size本身也没有多大，因此 $$3\times 3$$ 9个矩形中，每个矩形窗框都是可以感知到很大范围的。 

在划分为 $$n\times n$$ 个窗口后，我们把每个矩形窗口的中心点当成一个基准点，然后围绕这个基准点选取k(k=9)个不同scale、aspect&nbsp;ratio的anchor(论文中3个scale和3个aspect ratio)，对于每个anchor，首先在后面接上一个二分类softmax，有2个score输出用以表示其是一个物体的概率与不是一个物体的概率，然后再接上一个bounding box的regressor，以及4个坐标输出代表这个anchor的坐标位置，因此RPN的总体Loss函数可以定义为： 


  $$L(\{p_i\}\{t_i\}) = \frac{1}{N_{cls} }\sum_iL_{cls}(p_i, p^*_i)+\lambda\frac{1}{N_{reg} }\sum_ip^*_iL_{reg}(t_i, t^*_i)$$


这个公式里的 $$L_{reg}$$ 即为上面提到的 $$smooth L_1$$ 函数，而该项前面的 $$p^*_i$$ 表示这些regressor的loss指针对正样本而言，负样本的预测会直接舍去。 

另外在RPN训练中有一个需要注意的地方是正负样本的选择，文中提到如果对每幅图的所有anchor都去优化loss function，那么最终会因为负样本过多导致最终得到的模型对正样本预测准确率很低（It is possible to optimize&nbsp;for the loss functions of all anchors, but this will&nbsp;bias towards negative samples as they are dominate）。 

## Faster-RCNN训练步骤
  


说完了Fast-RCNN和RPN，现在是时候来讲Faster-RCNN最精华的部分了，也就是如何把这两者放在同一个网络结构中，如何训练出这样一个Multi-task的网络模型。 

我们知道，如果是分别训练两种不同任务的网络模型，即使它们的结构、参数完全一致，但各自的卷积层内的卷积核也会向着不同的方向改变，导致无法共享网络权重，Faster-RCNN提出了三种可能的方式： 

  1. Alternating training：此方法其实就是一个不断迭代的训练过程，既然分别训练RPN和Fast-RCNN可能让网络朝不同的方向收敛，那么我们可以先独立训练RPN，然后用这个RPN的网络权重对Fast-RCNN网络进行初始化，并且用之前RPN输出proposal作为此时Fast-RCNN的输入，之后不断迭代这个过程，即循环训练RPN、Fast-RCNN。 
  2. Approximate joint training：这里与前一种方法不同，不再是串行训练RPN和Fast-RCNN，而是尝试把二者融入到一个网络内，具体融合的网络结构如下图所示，可以看到，proposals是由中间的RPN层输出的，而不是从网络外部得到。需要注意的一点，名字中的"approximate"是因为&ldquo;this solution ignores the derivative w.r.t. the proposal&nbsp;boxes' coordinates that are also network responses&rdquo;，也就是说，反向传播阶段RPN产生的cls score能够获得梯度用以更新参数，但是proposal的坐标预测则直接把梯度舍弃了，这个设置可以使backward时该网络层能得到一个解析解（closed results），并且相对于Alternating traing减少了25-50%的训练时间。 
  3. Non-approximate training：上面的Approximate joint training把proposal的坐标预测梯度直接舍弃，所以被称作approximate，那么理论上如果不舍弃是不是能更好的提升RPN部分网络的性能呢？作者把这种训练方式称为&ldquo;&nbsp;Non-approximate joint training&rdquo;，但是此方法在paper中只是一笔带过，表示&ldquo;This is a nontrivial problem and&nbsp;&nbsp;a solution can be given by an &ldquo;RoI warping&rdquo; layer&nbsp;as developed in [15], which is beyond the scope of this paper&rdquo;，o(╯□╰)o 

<a href="/images/post/2016/03/FasterRCNN_train.png" rel="attachment wp-att-756" style="" target="" title=""><figure id="attachment_756" style="max-width: 288px" class="wp-caption aligncenter"><img src="/images/post/2016/03/FasterRCNN_train-288x300.png" alt="Approximate joint training" width="288" height="300" class="size-medium wp-image-756 wp-caption aligncenter" title="Approximate joint training" style="" srcset="/images/post/2016/03/FasterRCNN_train-288x300.png 288w, /images/post/2016/03/FasterRCNN_train.png 624w" sizes="(max-width: 288px) 100vw, 288px" /></figure></a> 

上面说完了三种可能的训练方法，可非常神奇的是作者发布的源代码里却傲娇的用了另外一种叫做4-Step Alternating Training的方法，思路和迭代的Alternating training有点类似，但是细节有点差别（rbg大神这样介绍训练方式我也是醉了），具体来说： 

  1. 第一步：用ImageNet模型初始化，独立训练一个RPN网络； 
  2. 第二步：仍然用ImageNet模型初始化，但是使用上一步RPN网络产生的proposal作为输入，训练一个Fast-RCNN网络，至此，两个网络每一层的参数完全不共享； 
  3. 第三步：使用第二步的Fast-RCNN网络参数初始化一个新的RPN网络，但是把RPN、Fast-RCNN共享的那些卷积层的learning&nbsp;rate设置为0，也就是不更新，仅仅更新RPN特有的那些网络层，重新训练，此时，两个网络已经共享了所有公共的卷积层； 
  4. 第四步：仍然固定共享的那些网络层，把Fast-RCNN特有的网络层也加入进来，形成一个unified network，继续训练，fine tune Fast-RCNN特有的网络层，此时，该网络已经实现我们设想的目标，即网络内部预测proposal并实现检测的功能。 

## 总结
  


至此，关于RCNN系列检测算法的关键部分已经全部介绍完毕，但应该还有很多真正的细节问题无法涉及到（其实也是因为好多细节实现我还没看o(╯□╰)o），从我个人感受而言，rbg真心碉堡，算是难得一个既能写paper又能coding的神人，而且最重要的一点，我的CVPR 2016论文里自己弄的一个数据集就借助了Fast-RCNN，否则应该没有可能写出这篇paper，在此再次跪谢rbg大神开放这么优秀的源代码造福我等低端代码搬运工！
