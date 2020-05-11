---
layout: post
title: centOS 编译最新版 ffmpeg (4.2.1) 动态库
date: 2020-05-11
categories:
- ffmpeg
tags: [linux,ffmpeg]
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

准备工作

mkdir $HOME/ffmepg_build  # 三方库安装目录
mkdir $HOME/ffmpeg_source # 所有库的下载目录
mkdir $HOME/bin           # 可执行二进制文件安装目录 

把安装的第三方库目录添加到系统查找路径中：

```
    sudo vim /etc/ld.so.conf
    # 添加, {$HOME} 替换成实际目录，比如 /home/slayer/ffmpeg_build/lib
    {$HOME}/ffmpeg_build/lib
    
    # 重新加载 ldconfig
    sudo ldconfig
```
### 编译依赖工具

``` 
sudo yum install automake autoconf libtool build-essential pkg-config gcc-c++
```

### 第三方依赖


-  nasm

    ``` 
    cd nasm
    ./autogen.sh
    PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin"
    PATH="$HOME/bin:$PATH" make
    make install
    ```

- yasm

   ``` 
   cd yasm-1.3.0 && ./configure --prefix="\(HOME/ffmpeg_build" --bindir="\)HOME/bin"
   make && make install
   ```

-  fdk-aac

   ``` 
   wget https://downloads.sourceforge.net/opencore-amr/fdk-aac-2.0.1.tar.gz
   tar xzvf fdk-aac-2.0.1.tar.gz 
   cd fdk-aac-2.0.1 && autoreconf -fiv && ./configure --prefix="$HOME/ffmpeg_build" --disable-shared
   make &&  make install
   ```

- opus

    ``` 
    cd opus-1.3.1 && ./configure --prefix="$HOME/ffmpeg_build" --enable-shared
    make && make install
    ```

- lame

   ```
   cd lame-3.00 && ./configure --prefix="$HOME/ffmpeg_build" --enable-nasm --enable-shared
   make &&  make install
   ```

- vorbis

    ``` 
    # install ogg
    sudo yum install libogg-devel.x86_64

    cd libvorbis && -- autoreconf -fi && ./autogen.sh

    ./configure --prefix="$HOME/ffmpeg_build" --enable-shared
    make 
    make install

    ```

-  x264

    ```
    PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --enable-shared 
    PATH="$HOME/bin:$PATH" make
    make install
    ```

- x265

    ```
    cd ~/ffmpeg_sources/x265/build/linux
    PATH="$HOME/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="$HOME/ffmpeg_build" -DENABLE_SHARED:bool=off  ../../source
    make && install
    ```

-  vpx

    ``` 
    PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --disable-examples --disable-unit-tests --enable-shared
    PATH="$HOME/bin:$PATH" make
    make install
    ```

- bluray

    

    ```
    # 安装 libxml2 
    sudo yum install  libxml2-devel.x86_64

    
    # 编译 bluray 
    wget https://download.videolan.org/pub/videolan/libbluray/1.1.2/libbluray-1.1.2.tar.bz2
    tar -jxvf libbluray-1.1.2.tar.bz2 
    cd libbluray-1.1.2/
    PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build"  --enable-shared --disable-bdjava-jar
    
    make && make install

    ```
  
- libxvid

    ```
    # 通过 yum 安装
    sudo yum install  xvidcore-devel.x86_64 

    
    
    # 手动安装
    wget http://packman.links2linux.de/download/xvidcore/3187274/libxvidcore4-1.3.5-pm152.2.6.x86_64.rpm
    wget http://packman.links2linux.de/download/xvidcore/3187274/libxvidcore4-1.3.5-pm152.2.6.x86_64.rpm
    // sudo yum localinstall libxvidcore4-1.3.5-pm152.2.6.x86_64.rpm 
    ```
    
- sdl2    

    ```
    wget https://www.libsdl.org/release/SDL2-2.0.12.tar.gz
    tar -xzvf SDL2-2.0.12.tar.gz
    cd SDL2-2.0.12/
    
    PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build"  --enable-shared 

    make -j8 && make install
    ```

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
    ./configure \
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

### 其他可选

- opencl

    系统默认没有opencl, 通常安装了显卡驱动都能加上 --enable-opencl

-  aom

    ``` 
    mkdir -p aom_build &&  cd aom_build

    PATH="$HOME/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="$HOME/ffmpeg_build" -DBUILD_SHARED_LIBS=1 -DENABLE_SHARED=on -DENABLE_NASM=on ../aom && \
    PATH="$HOME/bin:$PATH" make -j8  && \
    sudo make install
    ```
   
- freetype2

    ``` 
    cd freetype-2.10.1 && ./configure --prefix="$HOME/ffmpeg_build" --enable-shared
    make
    make install
    ```