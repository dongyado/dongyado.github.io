---
id: 895
title: 一个简单有效的判别性区域定位方法(Learning Deep Features for Discriminative Localization)
date: 2016-11-04T14:32:17+00:00
author: nicklhy
layout: post
guid: http://closure11.com/?p=895
permalink: '/%e4%b8%80%e4%b8%aa%e9%9d%9e%e5%b8%b8%e7%ae%80%e5%8d%95%e7%9b%b4%e6%8e%a5%e7%9a%84%e5%88%a4%e5%88%ab%e6%80%a7%e5%8c%ba%e5%9f%9f%e5%ae%9a%e4%bd%8d%e6%96%b9%e6%b3%95learning-deep-features-for-discrimina/'
views:
  - "343"
categories:
  - 计算机
---
前两天偶然看到一篇idea非常简单有效的用于定位图片判别性区域的paper，来自MIT大神[Bolei Zhou](http://people.csail.mit.edu/bzhou/)，做了一个简单的实现，发现确实work，在此做个简单分享。 

论文尝试解决的问题个人觉得非常有意义，就是在只有image-level label而没有类似detection、segmentation的标注时，我们该如何发现图片中对分类起决定性的那些判别区域。例如我们用ResNet对一张图片进行分类，输出结果显示有99%的概率这张图片里有一只熊猫，那么有没有办法可以告诉我们到底这只熊猫处在图片的哪个位置？ 

下图给出了一个更加具体的例子，即我们到底可以从图中哪里识别出&ldquo;刷牙&rdquo;、&ldquo;砍树&rdquo;的action？ 

[<img alt="screen-shot-2016-11-04-at-11-34-07-am" class="aligncenter size-full wp-image-896" height="512" src="http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-11.34.07-AM.png" width="960" srcset="http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-11.34.07-AM.png 960w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-11.34.07-AM-300x160.png 300w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-11.34.07-AM-768x410.png 768w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-11.34.07-AM-690x368.png 690w" sizes="(max-width: 960px) 100vw, 960px" />](http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-11.34.07-AM.png) 

&nbsp; 

## Class&nbsp;Activation Mapping
  


这篇论文所提出方法的核心思路在于通过CNN中最后一个卷积feature map后的Global Average Pooling(GAP)层，把那些用于Softmax分类但却已经失去spatial信息的一维特征向量在各个类别上的response映射回带有spatial信息的feature map上去。 

以GoogleNet为例，在用于Softmax分类的全连接层前，是一个能够把1024\*7\*7的feature map转化成1024维特征向量的GAP层，很显然这个特征向量通过softmax的全连接层后就可以得到原图片在各个类别上的响应。 

这里我们用 $$f_k(x, y)$$ 表示卷积feature map在坐标 $$(x, y)$$ 处、通道k的响应值，那么经过GAP后，通道k的响应值就变成了 $$F_k = \sum_{x, y}f_k(x, y)$$ ，再继续通过任意一个类别c的全连接层后 $$S_c=\sum_kw_k^cF_k$$ ，该类别最后的分类概率也就是 $$P_c = \frac{\exp{S_c} }{\sum_c\exp{S_c} }$$ ，注意，上述公式中的系数和bias项均被省略。 

现在如果我们再仔细分析一下这个 $$S_c$$ ，可以发现 


  $$S_c=\sum_kw_k^c\sum_{x, y}f_k(x, y)=\sum_{x, y}\sum_kw_k^cf_k(x, y)$$


注意，调换顺序后公式的含义是：整个图片的分类响应等于feature map各个空间位置特征的响应之和，而这些空间位置是可以映射回原图片的各个区域的，比如当前GoogleNet的feature map大小是7\*7，那么相当于是把原图分成7\*7的网格，分别求取每个网格对类别c的响应后再求和，只要能够计算出 $$M_c(x, y)=\sum_kw_k^cf_k(x, y)$$ ，也就相当于得到了图像各个区域的判别性热力图！下图对这一过程给出了更加形象的说明。 

[<img alt="screen-shot-2016-11-04-at-1-39-35-pm" class="aligncenter size-full wp-image-905" height="944" src="http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-1.39.35-PM.png" width="2028" srcset="http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-1.39.35-PM.png 2028w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-1.39.35-PM-300x140.png 300w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-1.39.35-PM-768x357.png 768w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-1.39.35-PM-1024x477.png 1024w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-1.39.35-PM-690x321.png 690w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-1.39.35-PM-980x456.png 980w" sizes="(max-width: 2028px) 100vw, 2028px" />](http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-04-at-1.39.35-PM.png)这个idea在实际实现时也非常简单，把图片forward一遍，然后再把分类权重和分类响应做个乘法即可，论文作者给了Matlab的代码，其中还包括一些其它的实验代码，可能看起来比较复杂，所以自己用MXNet对核心方法做了个简单实现，代码已提交到MXNet的master分支里，详情请点击[这里](https://github.com/dmlc/mxnet/blob/master/example/notebooks/class_active_maps.ipynb)。 

## Video CAM(Update)
  


今天又把这个idea在视频分类上做了一个应用，效果也非常不错，下面几幅图片展示了一些例子： 

[<img alt="baby_cam" class="aligncenter size-full wp-image-914" height="818" src="http://closure11.com/wp-content/uploads/2016/11/baby_cam.png" width="1970" srcset="http://closure11.com/wp-content/uploads/2016/11/baby_cam.png 1970w, http://closure11.com/wp-content/uploads/2016/11/baby_cam-300x125.png 300w, http://closure11.com/wp-content/uploads/2016/11/baby_cam-768x319.png 768w, http://closure11.com/wp-content/uploads/2016/11/baby_cam-1024x425.png 1024w, http://closure11.com/wp-content/uploads/2016/11/baby_cam-690x287.png 690w, http://closure11.com/wp-content/uploads/2016/11/baby_cam-980x407.png 980w" sizes="(max-width: 1970px) 100vw, 1970px" />](http://closure11.com/wp-content/uploads/2016/11/baby_cam.png) 

[<img alt="screen-shot-2016-11-09-at-1-45-46-pm" class="aligncenter size-full wp-image-910" height="1042" src="http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-09-at-1.45.46-PM.png" width="1988" srcset="http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-09-at-1.45.46-PM.png 1988w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-09-at-1.45.46-PM-300x157.png 300w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-09-at-1.45.46-PM-768x403.png 768w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-09-at-1.45.46-PM-1024x537.png 1024w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-09-at-1.45.46-PM-690x362.png 690w, http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-09-at-1.45.46-PM-980x514.png 980w" sizes="(max-width: 1988px) 100vw, 1988px" />](http://closure11.com/wp-content/uploads/2016/11/Screen-Shot-2016-11-09-at-1.45.46-PM.png) 

需要注意的是，这里所使用的视频分类网络是真正的video-level classification，而不是把每帧图片做图片分类，具体实现的方法与图片分类模型类似：在训练阶段，假设每个batch视频数量为N，每个视频所抽取的视频帧数量为L，特征维度为C，feature map分辨率为H*W，那么我们只需要把原来图片(N, C, H, W)三维的feature map换成视频(N, C, L, H, W)四维的feature map，再按照论文所述方法通过GAP转化成(N, C)的视频特征向量，接上一个softmax分类即可；在inference阶段（求CAM），同样用分类器weight与每个视频shape=(C, L, H, W)的feature map相乘即可获得其同时包含temporal和spatial信息的discriminative heat cube了。