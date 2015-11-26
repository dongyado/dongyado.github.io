---
layout: post
title: ubuntu下安装谷歌拼音（有些坑估计都踩过） 
date: 2015-05-23 22:50:04.000000000 +08:00
categories:
- linux
- tool
tags: [ubuntu]
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
description: '数一些安装google-pinyin的坑'

---

<!-- more -->
### 踩坑

在ubuntu安装google-pinyin的时候，通常会按照下面的步骤进行：

* 卸载 ibus
* 安装 fcitx输入法框架
* 安装谷歌拼音 fcitx-googlepinyin

可是，按照上面三步安装后，重启系统，一般会出现下面两个反应：

* 额，”系统设置“怎么不见了
* 输入法没启动～～

”系统设置“被牺牲了？因为在卸载ibus的时候，把ibus-gtk, ibus-gtk3等系统设置所需的东西给干掉了。

输入法没启动？ 因为你压根就没有把fcitx设置为系统默认输入法。

鉴于网上搜到的安装过程基本都是上面这三步，这就直接导致了一些用户踩坑。。

先说填坑的方法：

* 修复“系统设置”：
    
        sudo apt-get install unity-control-center

* 设置fcitx为系统默认输入法(二选一)：
    * System Settings->Language support->input Method 设置为fcitx
    * 执行下面命令，重新设置系统输入法，选中fcitx
    
            im-config

### 避免入坑

再说一下不入坑的安装方法，ibus没有必要卸掉，只需要改变系统默认方法就能切换，避免上述误伤的坑，如下：

* 安装输入法

        sudo add-apt-repository ppa:fcitx-team/nightly
        sudo apt-get update
        sudo apt-get install fcitx fcitx-googlepinyin

* 设置输入法，执行上面”设置fcitx为系统默认法“中的一个


然后重启，就ok了

如果不想重启，kill ibus-daemon && run fcitx

一般来说，这是不管用的。。。

### 如果桌面上什么都没有

说明是unity的配置文件出错了，把所有配置文件全部干掉并重启

        sudo rm -rf .config/compiz* .gconf/apps/compiz*
        killall gnome-session

恩，填坑结束，终于不用踩别人的坑了。


