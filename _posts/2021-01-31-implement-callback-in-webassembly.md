---
layout: post
title:  在 WebAssembly 中实现回调的方式
date: 2021-01-31
categories:
- webassembly
tags: [webassembly]
---

本文将介绍在 C++ 中实现 js 回调的几种方式. 在使用 wasm 的过程中, 避免不了要从 C++ 回调 js 的函数来实现异步交互.

官网文档 https://emscripten.org/docs/porting/connecting_cpp_and_javascript/Interacting-with-code.html#

中已经介绍了6种实现回调的方式, 这里介绍几种能解决实际问题的方式

---
### EM_ASM 相关参数介绍

EM_ASM 函数簇包含

```
EM_ASM
EM_ASM_INT
EM_ASM_DOUBLE
```

类似的使用方式 :

```
EM_ASM_({
            postMessage({cmd: 'callback', text: "callback", threadId: $2, callId : $0, code : $1})
        }, callback, code, tid);
```

其中 $0, $1, $2 分别代表 callback, code, tid 的值.

后面两个函数还可以获取到 js 返回的 int/ double 值. 一般能满足简单的使用.

---
### 如何在 worker 中实现回调

使用 wasm 的时候, 某些任务会被放到 worker 中执行, 执行完成后回调通知结果.

这个时候要特别注意: worker 和主线程是相互独立的, 并不能像普通的多线程可以共享进程内的数据. 在 worker 中调用 js 回调时, 第一个面临的限制就是 web worker 的限制:

1. 不能访问 window, document 对象
2. 与主线程通信需要通过 post message 方式
3. 不能访问主进程中的全局变量会对象

如果直接在 worker 中直接调用回调, 就只能做一些简单的事情. 为此 emscripten 提供了以下函数:

```
MAIN_THREAD_EM_ASM
MAIN_THREAD_EM_ASM_INT
MAIN_THREAD_EM_ASM_DOUBLE
MAIN_THREAD_ASYNC_EM_ASM
```

顾名思义就是在主线程中执行函数. 但是前面 3 个函数不管在哪个线程调用都会会阻塞主线程. 

推荐使用最后一个 MAIN_THREAD_ASYNC_EM_ASM, 区别就是在如果是主线程调用该函数,会被理解执行,如果是在其他线程调用,则会被追加到主线程的任务队列再被执行.

至于如何实现这个功能了, emscripten 也突破不了这个限制, 也是通过 postMessage 的通信机制实现的.

在不熟悉 emscripten 的时候我自己模拟了这个过程:

1. 在主线程注册函数, 保存在一个特定的对象中, 并产生一个 callid
2. 把callid 传到 worker 中,使用上述的 EM_ASM 回调
3. 子线程会把信息 post 到主线程
4. 主线程收到 message 和 callid, 去特定的对象查找到已经注册的回调函数, 执行函数,完成回调

### 如何在回调中传递字符串
在 emscripten 值的传递, 除了基本类型是通过拷贝的, 字符串和数组,内存都是通过指针地址去传递.

```
    std::string data = "{\"code\" : 0, \"msg\" : \"/input.mp4\"}"
    MAIN_THREAD_ASYNC_EM_ASM({
         var dataStr = UTF8ToString($0);
         console.log(dataStr)
    }, data.c_str());
```
在回调函数中,通过 UTF8ToString 函数把传递过去的指针转成 js 的字符串, 这样来达到传递字符串的目的.
