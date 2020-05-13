---
layout: post
title: CentOS 安装 nvidia 驱动, OpenGL
date: 2020-05-12
categories:
- linux
- video/audio tech
tags: [nvidia, opengl]
---

Ve SDK 需要兼容 nvidia 最新版 driver, nvcodec。特意组装了台式机，显卡为 gtx 1650。

需要的环境如下：

```
os: CentOS 7.5
gcc: 4.8.5.39
gpu driver: 440.82 显卡驱动
nvcodec: 9.2 视频硬编解码 SDK 
```

## 安装依赖

#### 安装内核构建依赖

nvidia 驱动在安装时需要重新构建内核以加载 gpu 驱动，需要安装 kernel-devel, kernel-headers

安装的 kernel-devel, kernel-headers 的版本要严格和当前内核版本一致。否则安装过程中会出现找不到 kernel headers 的错误

```
sudo yum -y insall gcc dkms epel-release

# 如果 yum 搜索到的 kernel-devel 和 kernel-headers 一致
sudo yum insall  kernel-devel  
sudo yum install kernel-headers

# 低版本比如 cenos 7.5 使用 yum 安装的版本与内核版本不一致
# 需要手动下载对应的版本安装，比如内核版本为 3.10.0-862

wget http://vault.centos.org/7.5.1804/os/x86_64/Packages/kernel-devel-3.10.0-862.el7.x86_64.rpm
wget http://vault.centos.org/7.5.1804/os/x86_64/Packages/kernel-headers-3.10.0-862.el7.x86_64.rpm

sudo yum install kernel-devel-3.10.0-862.el7.x86_64.rpm
sudo yum install kernel-headers-3.10.0-862.el7.x86_64.rpm
```

#### 禁用系统自带的 nouveau 驱动 (可选)

nouveau 是一个 nvidia 显卡的第三方开源驱动，一般 linux 发行版的内核都会默认自带并使用这个驱动模块。
但是一些新特性没有支持。

新版 nvidia 驱动安装时会自动禁用 nouveau 

查看 nouveau 有没有启用使用
```
lsmod | grep nouveau
```

禁用 nouveau；
```
sudo vim /etc/modprobe.d/blacklist-nouveau.conf

# 添加以下代码
blacklist nouveau
options nouveau modeset=0
```

## 安装驱动


#### 下载安装

```
wget https://us.download.nvidia.cn/XFree86/Linux-x86_64/440.82/NVIDIA-Linux-x86_64-440.82.run
sudo chmod +x NVIDIA-Linux-x86_64-440.82.run
sudo ./NVIDIA-Linux-x86_64-440.82.run
```

#### 重启检查是否安装成功

```
nvidia-smi
```


#### 检查 opengl 信息

nvidia 驱动自带 openGL 的实现，默认随着驱动被安装。

1. 使用 nvidia-settings 查看

```
$ nvidia-settings -g | grep -i "opengl"
OpenGL vendor string: NVIDIA Corporation
OpenGL renderer string: GeForce GTX 1650/PCIe/SSE2
OpenGL version string: 4.6.0 NVIDIA 440.82
OpenGL extensions:

```

2. 使用 mesa-utils 
```
$ sudo yum -y install mesa-utils
$ glxinfo | grep -i opengl
OpenGL vendor string: NVIDIA Corporation
OpenGL renderer string: GeForce GTX 1650/PCIe/SSE2
OpenGL core profile version string: 4.4.0 NVIDIA 440.82
OpenGL core profile shading language version string: 4.40 NVIDIA via Cg compiler
OpenGL core profile context flags: (none)
OpenGL core profile profile mask: core profile
OpenGL core profile extensions:
OpenGL version string: 4.6.0 NVIDIA 440.82
OpenGL shading language version string: 4.60 NVIDIA
OpenGL context flags: (none)
OpenGL profile mask: (none)
OpenGL extensions:
OpenGL ES profile version string: OpenGL ES 3.2 NVIDIA 440.82
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20
OpenGL ES profile extensions:
```


## 安装 cuda 驱动（可选)

```
wget http://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda_10.2.89_440.33.01_linux.run
chmod +x cuda_10.2.89_440.33.01_linux.run
sudo ./cuda_10.2.89_440.33.01_linux.run
```


### 其他实用命令

+ 查看显卡信息

```
$ sudo lshw -numeric -C display
  *-display UNCLAIMED       
       description: VGA compatible controller
       product: NVIDIA Corporation [10DE:1F82]
       vendor: NVIDIA Corporation [10DE]
       physical id: 0
       bus info: pci@0000:01:00.0
       version: a1
       width: 64 bits
       clock: 33MHz
       capabilities: pm msi pciexpress vga_controller bus_master cap_list
       configuration: latency=0
       resources: memory:f6000000-f6ffffff memory:e0000000-efffffff memory:f0000000-f1ffffff ioport:e000(size=128) memory:f7000000-f707ffff
```


或

```
$ lspci | grep -i nvidia
01:00.0 VGA compatible controller: NVIDIA Corporation Device 1f82 (rev a1)
01:00.1 Audio device: NVIDIA Corporation Device 10fa (rev a1)
```
