---
layout: post
title: ffmpeg sws_scale 颜色转换，右侧花屏问题研究及解决方案
date: 2019-12-20
categories:
- Ffmpeg
tags: [ffmpeg, sws_scale]
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  views: '2'
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---


使用 ffmpeg 解码视频后，需要将视频颜色从 yuv420p 转换到 rgba 后渲染。

在视频宽度不为8的倍数时，会出现视频画面右侧出现一条花屏。比如视频的宽度为 566， 566 % 8 = 6, 右侧会有6个像素宽度的花屏。


+ 转换代码如下：


```c
	// 初始化转换上下文对象，输入颜色模式为 yuv420p, 输出格式为 rgba, 宽高不变
	mColorConversionContext = sws_getCachedContext(	mColorConversionContext,
	                                                   mMeta.mWidth, mMeta.mHeight, AV_PIX_FMT_YUV420P,
	                                                   mMeta.mWidth, mMeta.mHeight, AV_PIX_FMT_RGBA,
													   SWS_FAST_BILINEAR, nullptr, nullptr, nullptr);
	
	// 初始化目标数据相关对象
	mConvertedFrame = av_frame_alloc();
	int bufferSize = av_image_get_buffer_size(AV_PIX_FMT_RGBA, mMeta.mWidth, mMeta.mHeight, 1);
	mConvertedBuffer = (uint8_t *)av_malloc(bufferSize * sizeof(uint8_t));
	av_image_fill_arrays(mConvertedFrame->data,
	                     mConvertedFrame->linesize,
	                     mConvertedBuffer,
	                     AV_PIX_FMT_RGBA,
	                     mMeta.mWidth,
	                     mMeta.mHeight, 1);
	// ...
	// 从 yuv420p 转换到 rgba
    sws_scale(mColorConversionContext,
              (const uint8_t *const *)mDecodeFrameContext->mFrame->data,
              mDecodeFrameContext->mFrame->linesize,
              0,
              mMeta.mHeight,
              mConvertedFrame->data,
              mConvertedFrame->linesize);


```

查找过程中，依次尝试了以下方法：
+ 直接把解码后的 AVFrame 写成图片，图片数据完好
+ 把 sws_scale 后的数据写入图片，图片右侧出现花屏

以此确定 sws_scale 转换后右侧超出的像素数据有问题，通过对目标数据内存填零（rgba 四位全部填0，则此时画面为透明），再把 sws_scale 转换后的数据写入图片，多出的部分为透明，
表明 sws_scale 并没有往那些位置填入颜色数据。

通过 ffmpeg 官方邮件用户组查到，该问题 2012 年就已经被发现了，直到 2019 年，正在使用的 4.1.3 版本也没有修复。 


+ 修复后的代码如下：

```c
	int swsFlags = SWS_FAST_BILINEAR;

	// 当视频宽高不为8的倍数，同时转换又没有尺寸变化时，会导致右侧出现一条花屏
	// 这个问题从 2012年被发现，到2019年的 4.1.3 版本还继续存在
	// 通过增加 SWS_ACCURATE_RND 可以规避这个问题
	if ((mMeta.mWidth & 0x7) || (mMeta.mHeight & 0x7)) {
		swsFlags |= SWS_ACCURATE_RND;
	}

	// 初始化转换上下文对象
	mColorConversionContext = sws_getCachedContext(	mColorConversionContext,
	                                                   mMeta.mWidth, mMeta.mHeight, AV_PIX_FMT_YUV420P,
	                                                   mMeta.mWidth, mMeta.mHeight, AV_PIX_FMT_RGBA,
													   swsFlags, nullptr, nullptr, nullptr);
	
	// ...

```

上述代码只在宽高补位8的倍数的情况下，启用 SWS_ACCURATE_RND 参数，强制 ffmpeg swscale 不使用 non_resizing 的算法，从而规避这个问题。

相关资料： http://libav-users.943685.n4.nabble.com/Libav-user-sws-scale-has-weird-behavior-when-not-resizing-td4655402.html







