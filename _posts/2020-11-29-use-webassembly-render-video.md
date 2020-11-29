---
layout: post
title:  WebAssembly 在 Web 端视频的应用
date: 2020-11-29
categories:
- webassembly
tags: [webassembly]
---


WebAssembly 从诞生起，赋予了前端更宽阔的应用想象。绘图视频渲染，剪辑，编解码，游戏都有可能基于 WebAssembly 在浏览器端推出相关的产品。  

### 什么是 WebAssembly

WebAssembly（wasm） 是一种二进制代码格式, 具有高效，跨平台性，包含这种格式的二进制文件，可以被各个平台的浏览器高效的加载，解析执行。  

只要浏览器支持 wasm， 用户便可以使用 wasm 所提供的功能，也就是说 wasm 的跨平台性其实是基于浏览器的跨平台性。上层用户编译 wasm 时，  
不需要关注底层架构是什么，只要编译出来正确的二进制文件，就可以在各个支持的浏览器运行。

wasm 增强了 js 的能力，js 不擅长做的事情，比如绘图，编码，解码，数学计算等，都可以在 wasm 中实现，然后 js 就可以使用wasm所提供的能力。  


现阶段已经有很多 WebAssembly 的应用，比如 ffmpeg 的编解码应用， unity 3d, unreal engine， google earth等都相继支持了 wasm。

### 如何使用 

WebAssembly 现在支持从 C/C++, go, rust 编译成 wasm 模块。使用 emscripten sdk, 可以从 C/C++ 源码直接编译成 wasm 文件，然后在网页中
直接加载使用。

参考 https://emscripten.org/docs/getting_started/Tutorial.html 入门。
参考 https://emscripten.org/docs/compiling/Building-Projects.html 编译项目。

但是如何从网页调用 wasm 的方法。两种方法：

+ 直接使用 emscripten 提供的底层函数，把wasm 的方法导出到 js 运行环境中。
+ 在底层使用 embinding 直接导出 c/c++ 的函数和类，然后在 js 中调用。

### 如果应用在视频剪辑中
基于 wasm 提供的能力，ffmpeg 解码， 图片解码， OpenGL, 多线程，视频剪辑应用的结构图如下， 该系列文章，会不定期更新，介绍基于此图的架构和技术原理：  

![image](/images/post/wasm.png)

参考在线演示 demo: http://cloudedit.atvideo.cc

### wasm 的限制和风险
wasm 的限制与不便
+ 运行在一个沙盒中，网页和 js 的限制， wasm 同样会有，比如跨域
+ 不能直接读取用户计算机磁盘上的文件
+ 能使用的最大内存有限制，各个浏览器的限制都不同，一般为 2G
+ 现阶段已经支持了多线程（基于 worker 和 SharedArrayBuffer), 其中 SharedArrayBuffer 在有些浏览器不会默认开启
+ wasm 中申请的内存也需要手动释放

技术本身不会有太多的风险，在于如何去使用。已经出现过基于 wasm 技术开发的挖矿程序，病毒等。  
由于 wasm 是更低级的二进制格式，网页端在这方面的安全检查和拦截还不成熟。现阶段使用了 wasm 技术的网站可能会有更大的安全风险。  

随着各大浏览器对 wasm 的完善， wasm 能支持的功能会更多。比如 gc, 更好的异常处理,届时能实现更高效和全面的功能。