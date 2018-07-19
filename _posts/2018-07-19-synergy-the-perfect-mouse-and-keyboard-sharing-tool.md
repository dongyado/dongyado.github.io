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

![server](/images/post/synerge-server.png)

第一次设置，需要设置好 IP，并且设置好电脑的显示器布局，synergy 会根据设置好的位置来进行鼠标的切换。

客户端

![server](/images/post/synerge-client.png)


### 已知 bug

1.9.0 以下的版本，在 mac 10.12 及以上的版本，在连接wifi工作时，会有严重的卡顿，
这个问题跟 mac 操作系统在 wifi 状态的管理有很大关系， synergy 在 1.9.0 修复了这个问题。

