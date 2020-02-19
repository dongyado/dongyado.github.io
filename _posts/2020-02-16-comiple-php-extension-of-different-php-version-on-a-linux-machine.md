---
layout: post
title: 在同一台linux机器编译多个 php 版本的扩展
date: 2020-02-16
categories:
- Ffmpeg
tags: [php, php_extension]
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

一般 php 扩展编译安装方式如下

### 扩展编译步骤 

以扩展 ataencoder 为例
+ 进入扩展目录
+ ./phpize
+ ./configure ./configure --enable-ataencoder
+ make 
+ make install


### 编译不同 php 版本的扩展

如果扩展需要兼容多个 php 版本，则需要在本地先编译测试，需要为不同版本编译不同的 so 包, 步骤如下:

### 安装不同版本的 php

+ 下载对应版本的源码，比如 7.0.33 和 7.2.10 版本
+ 分别进入两个版本的目录，编译并安装
+ 这个时候系统就有两个版本的 php 
+ 在 ubuntu 系统上，可以使用 update-alternatives 来设置默认的 php 版本，如果没有这个工具，则可以使用软链接实现

### 编译安装步骤
+ 选定要编译 php 版本的环境

需要保持　php, phpize, php-config 为目标版本。

phpize 用来初始化扩展，确定扩展使用的 php api 版本, 并且生成配置文件比如　configure 脚本  
php-config 被　configure 用来确定包含 php 的头文件和一些库

ubuntu 系统下可以使用　update-alternatives 替换, 其他 linux 发行版如果没有类似的工具，可以使用软链接替换。

+　设置 php 版本

```text
update-alternatives --list php
/usr/bin/php7.0
/usr/bin/php7.2

# 设置 php 为　7.0 版本
update-alternatives --set php /usr/bin/php7.0

# 或是使用软链接
# ln -s /usr/bin/php7.0 /usr/bin/php
```

+ 设置 phpize 版本

```text
# 查看可用的 phpize
update-alternatives --list phpize
/usr/bin/phpize7.0
/usr/bin/phpize7.2

# 设置 phpize 为　7.0 版本
update-alternatives --set phpize /usr/bin/phpize7.0

# 或是使用软链接
# ln -s /usr/bin/phpize7.0 /usr/bin/phpize
```

+ 设置 php-config 版本

```text
# 查看可用的 php-config
update-alternatives --list php-config
/usr/bin/php-config7.0
/usr/bin/php-config7.2

# 设置 php-config 为　7.0 版本
update-alternatives --set php-config /usr/bin/php-config7.0

# 或是使用软链接
# ln -s /usr/bin/php-config7.0 /usr/bin/php-config
```

设置完以上步骤后，就可以按文章开头的步骤编译安装 php7.0或其他 php 版本的扩展了。