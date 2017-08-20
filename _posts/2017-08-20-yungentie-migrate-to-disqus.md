---
layout: post
title: 将网易云跟帖评论导入 disqus 
date: 2017-08-20
categories:
- tools
tags: [大php, tools]
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

### 背景

两年前使用 jekyll+disqus  代替 wp 作为这个博客的系统的时候，把评论从 wp 导入了 disqus。  
以为从此就可以愉快的码字了，然而过了几个月，由于某种原因，disqus 说不能访问就不能访问了。  

在看过了国内几个第三方评论平台，最后投奔了多说，然后又因为某种原因，没用两个月，多说就关了。  

好吧，关了就关了吧，这不还有网易云跟帖嘛，花了点时间切到了云跟帖，云跟帖提供了导入多说评论的功能，  
就这样完美的导过去了，嗯，又能愉快的码字了。　  

然而，too young, 没用两个月，云跟帖也要关了。

到此，可以推测，国内的第三方评论平台全部被关掉只是时间问题。这时候q外的 disqus 反而是最稳定的平台了，  

毕竟程序员总有办法能访问。所以这些天计划切回 disqus。但是 disqus 并不支持从云跟帖导出的评论数据。
从这里反应出网易云跟帖的善后工作做的很不好，使用自定义的导出格式，给用户迁徙到其他平台造成了的很大的不方便。

最常见的应该是导出 wp 的格式，这样绝大部分平台都会支持导入。

既然 disqus 不支持，网上也没有轮子，那就只能自己造一个轮子了。

发这篇的博客的一个原因就是方便大家找到这个轮子，毕竟 github 比较难找到。

### 一个轮子，评论转换器

就一个 converter.php 文件，从网易云跟帖导出的评论文件读取数据，生成 disqus 支持的导入文件。

具体介绍参考下面的项目地址，项目有完整的原理和使用介绍。

[yungentie-migrate-to-disqus][] 

[yungentie-migrate-to-disqus]: https://github.com/dongyado/yungentie-migrate-to-disqus


