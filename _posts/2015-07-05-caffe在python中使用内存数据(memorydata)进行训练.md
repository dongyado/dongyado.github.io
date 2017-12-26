---
id: 672
title: Caffe在Python中使用内存数据(MemoryData)进行训练
date: 2015-07-05T13:49:03+00:00
author: nicklhy
layout: post
guid: http://closure11.com/?p=672
permalink: '/caffe%e5%9c%a8python%e4%b8%ad%e4%bd%bf%e7%94%a8%e5%86%85%e5%ad%98%e6%95%b0%e6%8d%aememorydata%e8%bf%9b%e8%a1%8c%e8%ae%ad%e7%bb%83/'
views:
  - "3311"
categories:
  - 计算机
---
最近在一个新的数据集（CompCar）上需要做很多层次的分类任务，做baseline时不可避免的会用到caffe，最开始时是用官网教程的方式先生成带标签的list列表，然后转成lmdb，最后caffe train --solver=xxx --weights=xxx这样进行训练，然而。。。实验做着做着硬盘就满了。。。原因一是因为数据集很大，二是在不同分类任务里要把图片列表分配成不同的标签，三是训练集测试集的split也有很多种，四是我还要把原始图片生成LMDB，整个过程算起来原始数据集已经增大了好几倍，后来想到了每次训练时不使用lmdb形式的数据，而改用内存数据，即手动读入图片列表中所有图片，然后做train、test的split，再分配label，这样只需在硬盘中保存最原始的图片文件和几个list列表文件即可，不过到网上搜了一下，关于Caffe的MemoryDataLayer、Python接口进行训练的资料居然非常少，也没有直观的例子。。。最后跟了一下caffe的各种PR加上fast-rcnn的代码，总算大概搞了明白，在此记录一下，代码可以参看<https://github.com/nicklhy/CompCar_Analysis>，仍在更新中（好吧，其实没多少代码）。 

一般做训练的话就是直接使用SGDSolver这个类了，它本身由C++实现的，是底层Net类的进一步封装，但是在Python接口中已经完美导出，可以很方便的使用。官方介绍非常简单&ldquo;<span style="color: rgb(0, 0, 0); font-family: Roboto, sans-serif; font-size: 14px; line-height: 22px;">Optimizes the parameters of a&nbsp;</span><a class="el" href="http://caffe.berkeleyvision.org/doxygen/classcaffe_1_1Net.html" style="color: rgb(70, 101, 162); font-weight: bold; text-decoration: none; font-family: Roboto, sans-serif; font-size: 14px; line-height: 22px;" title="Connects Layers together into a directed acyclic graph (DAG) specified by a NetParameter. ">Net</a><span style="color: rgb(0, 0, 0); font-family: Roboto, sans-serif; font-size: 14px; line-height: 22px;">&nbsp;using stochastic gradient descent (SGD) with momentum</span>&rdquo;，实际用也挺方便的： 

<pre class="brush:python;">import caffe

# 定义solver对象，solver_prototxt是模型定义文件的路径，即/path/to/xxx_solver.prototxt
solver = caffe.SGDSolver(solver_prototxt)

# 如果存在预训练的权重，则载入
if pretrained_model is not None:
    solver.net.copy_from(pretrained_model)

while True:
    # 准备数据(如果模型定义的是MemoryData)
    solver.net.set_input_arrays(train_X, train_Y)
    # 迭代
    solver.step(1)
</pre>

但是实际在写代码时有一些关键点需要注意： 

  1. caffe.set_device一定要放在最前面，不要等Net或者SGDSolver变量都定义完了以后在执行。。。（悲伤的debug回忆） 
  2. 如果模型定义时有区分training和validation的不同phase，那么在solver中实际上是存在两个表示网络的成员变量：solver.net和solver.test\_nets，注意，前者直接就是一个Net的对象，而后者是Net对象的列表，如果像GoogleNet那样，存在一个training和一个testing(validation而不是真正的testing，做测试的文件其实是deploy.prototxt)，那么应该通过solver.test\_nets[0]来引用这个测试网络；另外，测试网络和训练网络应该是共享中间的特征网络层权重，只有那些标出include { phase: TRAIN }或者<span style="line-height: 20.7999992370605px;">include { phase: TEST }的网络层有区分；</span> 
  3. 训练数据train\_X, train\_Y必须是numpy中的float32浮点矩阵，train\_X维度是sample\_num\*channels\*height*width，train\_Y是sample\_num维度的label向量，这里sample\_num必须是trainning输入batch\_size的整数倍，为了方便，我在实际使用时每次迭代只在整个训练集中随机选取一个batch_size的图片数据放进去； 
  4. solver.step(1)即迭代一次，包括了forward和backward，solver.iter标识了当前的迭代次数； 
  5. 如果在solver.prototxt中设置了test\_interval，那么在solver.test\_interval的整数倍迭代时需要做一次完整的测试，此时设置数据应该是 <pre class="brush:python;">solver.test_nets[0].set_input_arrays(test_X, test_Y)</pre>
    
    之后计算loss或者准确率时也不能使用solver.step函数，应为测试不需要backward过程 
    
    <pre class="brush:python;">solver.test_nets[0].forward()</pre>
    
    这之后就可以访问solver.test\_nets[0].blobs['blob\_name'].data来获取loss或者准确率了。 

  6. solver.prototxt文件中的参数可以通过google.protobuf这个模块实现 
    
    <pre class="brush:python;">import caffe
import google.protobuf as pb2

solver_param = caffe.proto.caffe_pb2.SolverParameter()
with open(solver_prototxt, &#39;rt&#39;) as fd:
    pb2.text_format.Merge(fd.read(), solver_param)</pre>
    
    其中solver\_param首先从caffe.proto.caffe\_pb2.SolverParameter()获取默认参数，然后接下来从solver.prototxt文件里获取实际定义的参数，之后在引用时就可以很方便的看到诸如学习率、测试周期等变量了 

  7. 在通过Python接口设置MemoryData时有时候会遇到一个奇怪的问题 
    
    <pre class="brush:bash;">Segmentation faults and Check failed: status == CUBLAS_STATUS_SUCCESS (14 vs. 0) CUBLAS_STATUS_INTERNAL_ERROR.</pre>
    
    这个在Google了之后看到了<https://github.com/BVLC/caffe/issues/2334>这个帖子，题主说他已经解决了这个问题，通过在数据传输时使用deep copy，并且提交了一个[pr](https://github.com/TJKlein/caffe/commit/5f1bb97a587043dbe0892466b866abfe4c76804c)，测试了一下居然真的可以，但奇怪的是这个request还没有合并到master分支里。。。不知道是不是因为Python的这个接口用的人少还是什么情况。 

  8. 需要保存模型时 
    
    <pre class="brush:python;">solver.net.save(model_path)</pre>
    
    此时会生成一个caffemodel和一个solverstate文件。 

  9. 学习率及更新方式等参数只要在solver.prototxt中已经设置，不需要在代码中手动设置，比如我在solver.prototxt里设置了base\_lr是0.005，lr\_policy是"step"，stepsize是6000，gama是0.7，那么只要你不停的step，在6000次以后学习率就会自动变成0.005*0.7。 

 10. 如果在solver.prototxt文件里有average\_loss这个参数（比如GoogleNet），也就是说要取多次forward操作的loss做平均，那么在set\_input\_arrays时记得输入average\_loss*training\_batch\_size这么多的训练数据，然后调用solver.step(average\_loss)，如果仍然是每次step一次，那么average\_loss参数就会失效，相当于没有起作用，具体实现可以在Caffe的源代码里找到，主要是memory\_data\_layer.cpp文件里： 
    
    <pre class="brush:cpp;">template &lt;typename Dtype&gt;
 void MemoryDataLayer&lt;Dtype&gt;::Forward_cpu(const vector&lt;Blob&lt;Dtype&gt;*&gt;& bottom,
     &brvbar; const vector&lt;Blob&lt;Dtype&gt;*&gt;& top) {
   CHECK(data_) &lt;&lt; "MemoryDataLayer needs to be initalized by calling Reset";
   top[0]-&gt;Reshape(batch_size_, channels_, height_, width_);
   top[1]-&gt;Reshape(batch_size_, 1, 1, 1); 
   top[0]-&gt;set_cpu_data(data_ + pos_ * size_);
   top[1]-&gt;set_cpu_data(labels_ + pos_);
   pos_ = (pos_ + batch_size_) % n_; 
   if (pos_ == 0)
     has_new_data_ = false;
}</pre>
    
    如果之前通过Reset函数设置的data shape是多个batch\_size，那么每次Step时就会调用到pos\_ = (pos\_+batch\_size\_)%n，不断feed下一个batch\_size的数据。 

完整代码见<https://github.com/nicklhy/CompCar_Analysis>中的<span style="font-family:monospace"><span style="color: rgb(0, 0, 0);">src/caffe/train.py文件。</span></span>