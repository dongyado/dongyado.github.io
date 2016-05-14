---
layout: post
title: 内网机器通过脚本获取tplink路由器的外网IP
date: 2016-05-14
categories:
- remote-control-door
- tool
- funny
tags: [funny,toss]
status: publish
type: post
published: true
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---

一般来说，子网机器是无法直接拿到路由器的IP，
就比如我的笔记本是没法直接获取所连接路由器的IP，只能登录路由器控制台才能查看到分配给路由器的IP。

之前有一篇博客总结了几种内网机器获取路由器外网IP的方法，查看[several-ways-to-get-router-external-ip][]。

这里实现第三种方法，使用脚本去获取。

### 思路
既然可以在本机登录路由器控制面板，并能看到外网IP，那肯定能通过脚本去模拟登录。
不过各个路由器厂商的登录方式都不一样，需要在浏览器中使用开发者工具去调试。

通过断点调试发现，每隔几秒就有请求发出：

  /?code=2&asyn=1&id=R0TPpo5pDg%3CJr)5k

header 里面还传了一个request payload, 数据只
返回的信息：
  
  00000 id 23 ip 219.133.xxx.xxx mask 255.255.255.255 gateway 219.133.xxx.xxx dns 0 202.96.128.86 dns 1 202.96.134.133 status 1 code 0 upTime 0 inPkts 2147576644 inOctets 2 outPkts 2149255756 outOctets 0



[several-ways-to-get-router-external-ip]:/tool/funny/2016/04/17/several-ways-to-get-router-external-ip/