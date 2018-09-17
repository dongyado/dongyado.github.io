---
layout: post
title: Synergy —— 一套键盘鼠标控制多台电脑(支持linux, win, mac)
date: 2018-07-19
categories:
- tool
tags: [Synergy]
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

### 为啥会有这个需求
如果你工作的时候有两台电脑，一台用来办公，一台用来沟通或者看资料。
一般情况就是，这两台电脑分别有一套键盘和鼠标，需要用哪一台的电脑的时候，
就用那一台的键盘鼠标操作。这样除了多了一套键鼠占用小小办公桌的一块地方之外，
来回在两套键鼠上面操作，也很不方便。

于是，使用一套键鼠控制两台或多台电脑的需求就有了，各种各样的键鼠共享的硬件和
软件就都出来了。试用了几款软件，推荐 synergy，使用了一年左右，一直稳定运行。

### Synergy 

Synergy 支持 linux, win, mac。工作原理就是：需要共享键盘鼠标的电脑，都必须链接在同一个局域网下，
每台电脑装一个 Synergy, 可以设置为服务器模式和客户端模式运行。键盘鼠标实际连接的电脑作为服务器模式运行，
没有键盘鼠标的电脑都设置为客户端模式运行。服务端捕获主机的键盘和鼠标事件，然后通过路由器发送给客户端电脑，
从而实现键盘鼠标的共享。


代码托管在 github ： https://github.com/symless/synergy-core

官网：https://symless.com/synergy


### 安装使用
 
官网必须购买才能下载，有两种方法可以安装试用：

+　github 上面拉下代码安装

发布的版本：　https://github.com/symless/synergy-core/releases
根据下面的教程编译安装： https://github.com/symless/synergy-core/wiki/Compiling

+ 下载发布出来的老版本

https://sourceforge.net/projects/synergy-stable-builds/files/?source=navbar

### 设置图
服务端

![server](/images/post/synergy-server.jpg)

第一次设置，需要设置好 IP，并且设置好电脑的显示器布局，synergy 会根据设置好的位置来进行鼠标的切换。

客户端

![client](/images/post/synergy-client.jpg)


### 已知 bug

1.9.0 以下的版本，在 mac 10.12 及以上的版本，在连接wifi工作时，会有严重的卡顿，
这个问题跟 mac 操作系统在 wifi 状态的管理有很大关系， synergy 在 1.9.0 修复了这个问题。

### mac 系统，打开应用软件无限崩溃问题

今天重启了一下系统，然后 synergy 打开就直接崩溃了，重装后打开还是崩。

初步看了下 log ：

```text
Exception Type:        EXC_BAD_ACCESS (SIGSEGV)
Exception Codes:       KERN_INVALID_ADDRESS at 0x0000000000000010
Exception Note:        EXC_CORPSE_NOTIFY

Termination Signal:    Segmentation fault: 11
Termination Reason:    Namespace SIGNAL, Code 0xb
Terminating Process:   exc handler [0]

```

遇到这种问题一般有几个原因：

+ 配置文件保存失败或者格式有误，应用在启动的时候没有做好容错处理
+ 应用退出保存的运行数据有问题，应用在重新打开时会去读取已经存在的运行数据导致崩溃
+ 系统版本与应用版本不兼容(系统版本和应用版本都未改变，排除该原因)

查了一下大家解决的办法基本就是卸载重装，但是大概率并不能解决问题。
因为卸载的时候不一定能保证该应用的配置文件和运行数据被移除感觉，特别是在 mac 系统，
从 applications 中卸载应用也就是把应用相关的文件移除了，配置文件和运行数据压根就不会动。  

所以不管你卸载重装多次，如果是由配置文件和运行数据导致的崩溃，都无法解决。

所以最关键的是找到该应用启动的时候打开了哪些文件，我总结了以下三种方法：


+ 查看 mac 系统中该应用的配置文件和运行数据的路径，配置文件一般会在用户目录下，而数据文件会在
    用户目录下的 Library/Saved Application State/　里面

+ 如果奔溃的进程还存在，通过以下两种方法查看锁打开的文件

    - 通过进程查看器查看打开的文件
        
        ```text
        ...
        /Users/dongya/Library/Saved Application State/synergy.savedState/data.data
        /System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/HIToolbox.framework/Versions/A/Resources/Extras2.rsrc
        /System/Library/Frameworks/CoreImage.framework/Versions/A/Resources/ci_kernels.metallib
        /private/var/folders/q4/hbvrj59n1ps_h9q30hcfpq_00000gn/C/synergy/com.apple.metal/libraries.maps
        /private/var/folders/q4/hbvrj59n1ps_h9q30hcfpq_00000gn/C/synergy/com.apple.metal/libraries.data
        ...
        ```
        
    - 通过 lsof -p 命名查看打开的文件
        
        ```text
        ➜  /var lsof -p 1664
        COMMAND   PID   USER   NAME
        synergyc 1664 dongya   /
        synergyc 1664 dongya   /Applications/Synergy.app/Contents/MacOS/synergyc
        synergyc 1664 dongya   /private/var/db/timezone/tz/2018e.1.0/icutz/icutz44l.dat
        synergyc 1664 dongya   /System/Library/Caches/com.apple.IntlDataCache.le.kbdx
        synergyc 1664 dongya   /usr/share/icu/icudt59l.dat
        synergyc 1664 dongya   /System/Library/CoreServices/SystemAppearance.bundle/Contents/Resources/SystemAppearance.car
        synergyc 1664 dongya   /private/var/folders/q4/hbvrj59n1ps_h9q30hcfpq_00000gn/0/com.apple.LaunchServices-221-v2.csstore
        synergyc 1664 dongya   /System/Library/Keyboard Layouts/AppleKeyboardLayouts.bundle/Contents/Resources/AppleKeyboardLayouts-L.dat
        synergyc 1664 dongya   /usr/lib/dyld
        synergyc 1664 dongya   /private/var/db/dyld/dyld_shared_cache_x86_64h
        synergyc 1664 dongya   ->0xea113beb9b79d7b9
        synergyc 1664 dongya   ->0xea113beb9b79e179
        synergyc 1664 dongya   ->0xea113beb9b79c2b9
        synergyc 1664 dongya   /Users/dongya/Library/Saved Application State/synergy.savedState/data.data
        synergyc 1664 dongya   /System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/HIToolbox.framework/Versions/A/Resources/Extras2.rsrc

        ```
    

+ 查看崩溃日志看是否能找到一些文件路径


通过上面的方法找出遗留的文件，如果可以修复就修复，不能修复直接删除，再重装的对应的应用即可。

上面的思路也可以作为解决 mac 系统下应用无限崩溃的通用方法之一。




