---
layout: post
title: 如何高效的调试 ffmpeg 
date: 2020-05-24
categories:
- ffmpeg
tags: [ffmpeg]
---
### 为什么需要调试 ffmpeg 

Ffmpeg 作为音频编解码封装最流行的基础工具，也是一个庞大的程序，大部分使用是使用命令行去处理。但是也有更高级的应用，比如在手机端，服务端开发基于 ffmpeg 开放的 api 的应用程序。

在移动端，特别是安卓端，只要有音视频的处理，基本是离不开 ffmpeg 开放的 api 。在服务端进行视频渲染解码，封装转码也需要ffmpeg。

但是在熟悉精通整个 ffmpeg 的大部分接口和音视频相关基础之前，几乎所有人都会遇到 ffmpeg 接口返回的错误码，比如 -1, -22。一般的处理情况都是去看 ffmpeg 提供的 examples, 或者再次搜索一下其他人有没有遇到同样的问题。

但是 ffmpeg 返回的错误码是公用的，比如 -22 表示 invalid argument, 但是到底是哪一个参数没设置正确， ffmpeg 根本没有任何提示。对于这种问题，供刚入门 ffmpeg 的童鞋的解决方法，要么用排除法，一个一个的方式去试，幸运的时候能试出来，但是试不出来是常态，可能会在这样一个问题上浪费几个小时，甚至几天的时间。

所以需要找到更高效的方法。

### 如何高效调试

#### 1. 让  ffmpeg  打印 debug 级别的日志

```
# 在调用 ffmpeg 相关函数前 设置 ffmpeg 打印的日志级别为 debug
av_log_set_level(AV_LOG_DEBUG);
```

在运行程序时， ffmpeg 会打印所有调试信息，这时候就是要找到可能的反应原因的错误信息，但是我们也有很大可能还是找不到具体原因。

#### 2. 编译带完整debug 信息的 ffmpeg 
在使用 ffmpeg 接口出现了问题时，如果能通过 gdb 断点进去 ffmepg 相关的文件去调试，这对有助于快速定位问题。

但是需要我们手动编译带 debug 信息，并且去掉编译优化的 ffmpeg 库：

```
./configure \
    --prefix="/usr/" \
    --pkg-config-flags="--static" \
    --extra-cflags="-I$HOME/ffmpeg_build/include" \
    --extra-ldflags="-L$HOME/ffmpeg_build/lib" \
    --extra-libs="-lpthread -lm" \
    --bindir="/usr/bin" \
    --enable-debug=3\
    --disable-optimizations \
    --disable-stripping \
    --enable-shared \
    --enable-pic \
    --enable-gpl \
    --enable-nonfree \
    ...
```
--disable-optimizations 用于去掉编译的优化，这样可以避免在 gdb 调试时，变量出现 `optimized out` 的提示

--disable-stripping 禁止去掉 gdb 所需的符号信息

编译出来的库，就可以使用 gdb 跟踪具体在 ffmpeg 中的哪一步出错，能快速定位问题。