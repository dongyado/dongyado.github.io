---
layout: post
title: MacOS 编译最新版 ffmpeg (4.2.1) 动态库
date: 2020-05-11
categories:
- ffmpeg
- video/audio tech
tags: [macos,ffmpeg]
---

在 macOS 下编译相对于 linux 要简单一些。由于在 macOS 下一般是开发使用，所以在编译之前使用 homebrew 安装相关依赖即可。


### 安装依赖
- automake 1.16.2

    brew install automake

- pkg-config 0.29.2

    brew install pkg-config

- fdk-aac 2.0.1 

    brew install fdk-aac

- lame 3.100

    brew install lame

- libvorbis 1.3.6 

    brew install libvorbis

- libopus 1.3.1

    brew install opus

- libbluray 1.1.1

    brew install libbluray

- libvpx 1.8.2

    brew install libvpx

-  libx265 3.3

    brew install x265

-  libx264 r2917

    brew install x264

-  libxvid 1.3.5

    brew install xvid

- sdl2   2.0.9

    brew install sdl2

### 2. 编译 ffmpeg

- 下载
    
    ``` 
    wget http://ffmpeg.org/releases/ffmpeg-4.2.1.tar.gz
    tar -xzvf ffmpeg-4.2.1.tar.gz
    cd ffmpeg-4.2.1
    ```


- 执行 ./configure 检查依赖，出现 not found 请检查，依赖库是否有安装成功

    以下 enable 的选项是 ve SDK 所需的，其他选项可自行选择添加

    ```
    ./configure  \
    --prefix=/usr/local \
    --enable-shared \
    --enable-pic \
    --enable-gpl \
    --enable-nonfree \
    --enable-libmp3lame \
    --enable-libfdk_aac \
    --enable-libvorbis \
    --enable-libopus \
    --enable-libbluray \
    --enable-libvpx \
    --enable-libx265 \
    --enable-libx264 \
    --enable-libxvid \
    --enable-lzma \
    --enable-opencl \
    --enable-audiotoolbox \
    --enable-videotoolbox \
    --enable-sdl2 \
    --enable-pthreads \
    --enable-x86asm \
    --enable-postproc \
    --disable-securetransport \
    --disable-libjack \
    --disable-libopencore-amrnb \
    --disable-libopencore-amrwb \
    --disable-libxcb \
    --disable-libxcb-shm \
    --disable-libxcb-xfixes \
    --disable-indevs \
    --disable-outdevs 
    ```

- 编译安装

    ``` 
    make -j8 && make install
    ```
        
        

--- 
###  ffmpeg 非必须选项, 按需选择

-  libmodplug 0.8.9.0

    brew install libmodplug

-  libopenjpeg 2.3.1

    brew install openjpeg

-  libsoxr 0.1.3

    brew install libsoxr

- libspeex  1.2.0

    brew install speex

- gnutls 3.6.8 

    brew install gnutls
