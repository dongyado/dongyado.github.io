---
layout: post
title: 中文分词器 friso，以及开始支持 php7 扩展
date: 2017-08-20
categories:
- tool
tags: [中文分词器, friso, php]
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

### 什么是 friso

Friso是使用c语言开发的一款开源的高性能中文分词器，使用流行的mmseg算法实现。  
完全基于模块化设计和实现，可以很方便的植入其他程序中，例如：MySQL，PHP，  
源码无需修改就能在各种平台下编译使用，加载完20万的词条，内存占用稳定为14.5M.

要了解 mmseg 分词算法，请点击 [mmseg][]

### friso 的特性

+ 三种切分模式：

    - 简易模式：FMM算法，适合速度要求场合。
    - 复杂模式- MMSEG四种过滤算法，具有较高的岐义去除，分词准确率达到了98.41%。
    - (!New)检测模式：只返回词库中已有的词条，很适合某些应用场合。(1.6.1版本开始)


+ 支持自定义词库。在dict文件夹下，可以随便添加/删除/更改词库和词库词条，并且对词库进行了分类。

+ 简体/繁体/简体混合支持, 可以方便的针对简体，繁体或者简繁体切分。同时还可以以此实现简繁体的相互检索。

+ 支持中英/英中混合词的识别(维护词库可以识别任何一种组合)。例如：卡拉ok, 漂亮mm, c语言，IC卡，哆啦a梦。

+ 很好的英文支持，英文标点组合词识别, 例如c++, c#, 电子邮件，网址，小数，百分数。

+ (!New)自定义保留标点：你可以自定义保留在切分结果中的标点，这样可以识别出一些复杂的组合，例如：c++, k&r，code.google.com。

+ (!New)复杂英文切分的二次切分：默认Friso会保留数字和字母的原组合，开启此功能，可以进行二次切分提高检索的命中率。例如：qq2013会被切分成：qq/ 2013/ qq2013。

+ 支持阿拉伯数字/小数基本单字单位的识别，例如2012年，1.75米，5吨，120斤，38.6℃。

+ 自动英文圆角/半角，大写/小写转换。

+ 同义词匹配：自动中文/英文同义词追加. (需要在friso.ini中开启friso.add_syn选项)。

+ 自动中英文停止词过滤。(需要在friso.ini中开启friso.clr_stw选项)。

+ 多配置支持, 安全的应用于多进程/多线程环境。

+ 提供friso.ini配置文件, 可以依据你的需求轻松打造适合于你的应用的分词。


### 分词速度

测试环境：2.8GHZ/2G/Ubuntu

简单模式：3.8M/秒

复杂模式：1.8M/秒


### friso 支持的客户端

开发 friso 的最初目的就是为 php 提供一个高性能的中文分词器，所以对 php 的支持是最好的。

现已支持：

+ 命令行查询
+ php5 扩展
+ php7 扩展（2017-08-18开始支持）
+ sphinx

### 项目地址


[friso@github][] 
[friso@osc][]


[friso@github]: https://github.com/lionsoul2014/friso
[friso@osc]: https://gitee.com/lionsoul/friso

[mmseg]: http://technology.chtsai.org/mmseg/%E3%80%82


