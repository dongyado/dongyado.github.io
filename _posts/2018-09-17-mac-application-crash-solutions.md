---
layout: post
title: Mac 系统，应用突然无限崩溃的一般原因和解决方法
date: 2018-09-17
categories:
- 随笔
tags: [mac, synergy]
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

今天无意重启了一下系统，再打开 Synergy 就直接崩溃，重装后打开还是崩。

查看日志有以下信息：

```text
Exception Type:        EXC_BAD_ACCESS (SIGSEGV)
Exception Codes:       KERN_INVALID_ADDRESS at 0x0000000000000010
Exception Note:        EXC_CORPSE_NOTIFY

Termination Signal:    Segmentation fault: 11
Termination Reason:    Namespace SIGNAL, Code 0xb
Terminating Process:   exc handler [0]

```

日志表明内存访问错误，原因可能有多种，因为之前是正常使用了很久，系统版本，
应用版本都没有变化，所以原因肯定不在系统和应用程序本身，可能下面两个原因：

+ 配置文件保存失败或者格式有误，应用在启动的时候没有做好容错处理
+ 应用退出保存的运行数据有问题，应用在重新打开时会去读取已经存在的运行数据导致崩溃


事先查了一下网上解决这个问题的方法基本就是卸载重装，但是并不能解决这个问题。
因为卸载的时候不一定能该应用的配置文件和运行数据被移除干净，特别是在 mac 系统，
从 applications 中卸载应用也就是把应用相关的安装程序文件家移除了，配置文件和运行数据压根就不会动。  


所以最关键的是找到该应用启动的时候打开了哪些配置和数据文件，有以下以下三种方法：


+ 查看 mac 系统中该应用的配置文件和运行数据的路径，配置文件一般会在用户目录下，而数据文件会在
    用户目录下的 Library/Saved Application State/　里

+ 如果奔溃的进程还存在，通过以下两种方法查看锁打开的文件

    mac的进程崩溃后，有些时候并没有退出，所以才能查看该进程的信息

    - 通过“活动监视器”查看打开的文件
    
        ```text
        活动监视器 -> 搜索找到对应进程 -> 双击弹出进程的详细信息 -> 点击“打开的文件和端口”
        ```
        
        可以看考类似以下的信息，第一行就是 Synergy 保存的运行数据：
        
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


通过上面方法找出相关的文件，可以修复就修复，不能修复直接删除，再重装的对应的应用即可。
