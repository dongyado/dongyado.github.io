---
layout: post
title: linux常用命令备忘
date: 2014-10-27 18:04:48.000000000 +08:00
categories:
- linux
- tool
tags: [linux]
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
  first_name: ''
  last_name: ''
---

<!-- more -->

#### 1. 删除过期文件

    find /tmp/ -type f -mtime +1 -exec rm -f {} 

/tmp/是路径
-type 是文件类型 f 表示普通文件
-mtime 表示最后修改时间 +1 表示一天之前
-exec 表示执行命令 rm -f
{} 表示查找到的文件名

注意

    find /tmp/ -type f -mtime +1 -exec rm -f {} \;  # {} 与 \之间必须有一个空格

结合crontab进行定时清理
     0 4 * * * find /tmp/ -type f -mtime +1 -exec rm -f {} \; 

表示每天凌晨4点清理。


#### 2. 网络带宽查看

*   nload 很好的一个工具，按TAB键切换网卡
*   bmon 提供了很详细的带宽信息
