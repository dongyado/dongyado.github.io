---
layout: post
title: 使用ionic开发自己的app 
date: 2015-05-24 13:50:04.000000000 +08:00
categories:
- 前端技术
- app
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

description: 'ionic是一个用来开发混合手机应用的，开源的，免费的代码库。可以优化html、css和js的性能，构建高效的应用程序，而且还可以用于构建Sass和AngularJS的优化。ionic会是一个可以信赖的框架。'

---

<!-- more -->
### ionic是什么？

ionic是一个用来开发混合手机应用的，开源的，免费的代码库。可以优化html、css和js的性能，构建高效的应用程序，而且还可以用于构建Sass和AngularJS的优化。ionic会是一个可以信赖的框架。

其实就是使用js,css,html开发app的一个框架，在使用的过程中，发现ionic的优点很多：

* 基于angularjs，sass，开发便捷，速度快
* 文档很完善，社区也很活跃
* 命令行工具相当完善，提供了很多功能
* 集成了crosswalk(提供webkit webview代替系统自带webview)后，性能有几倍的提升
* 在ios下性能很好（主要是ios系统就优化的很好），不需要集成crosswalk

缺点:

* 在android上性能不佳，主要原因是android自带的浏览器太渣，将crosswalk集成后，性能与ios无太大差别
* 1.0版本才发布两个多月，这可以说也是个优点
* 因为基于angularjs， 所以之前没有接触过angular的需要熟悉一下


### 为什么要使用ionic?

没接触过android,ios原生应用开发，没有时间，精力去学习，但是为了快速开发自己的app，这时候ionic就能很好的帮助我们，可以在一两周内完成自己的app。

### 安装ionic

* 安装所需环境，node.js、npm
* 安装cordova(不过多介绍，自行脑补)，ionic
    
        sudo npm install -g cordova ionic

* 初始化一个名为myApp的项目，项目名后面可以指定以哪种方式初始化（blank|tabs|sidemenu）

        ionic start myApp tabs

* 运行App， 会在浏览器中打开应用
    
        cd myApp && ionic setup sass && ionic serve

### 安装打包环境
    
ionic需要配置android打包环境（ios也是如此）:

* 下载jdk
* 下载android sdk tools, 下载链接：http://developer.android.com/sdk/installing/index.html
* 配置jdk和android sdk tools，vim /etc/profile, 添加

        # for jdk
        export JAVA_HOME={your-jdk-directory}
        export JRE_HOME=${JAVA_HOME}/jre
        export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
        export PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin:$PATH

        # for android
        export ANDROID_HOME={your-android-sdk-directory}
        export PATH=${PATH}:ANDROID_HOME/platform-tools:ANDROID_HOME/tools

* 执行下面命令检查环境配置是否正确,找不到命令，说明配置不成功

        source /etc/profile

        java -version # 是否能打印jdk的信息

        android       # 是否能打开android sdk manager


* 执行 android命令打开android sdk manager，下载需要的sdk api, build tools
    
    * 选择 API 22和最新的build tools，并点击install packages.（要翻墙）
    * 执行 android avd 设置安卓模拟器
    * 设置 API level and other config.

### 一些坑

* 如果是64位的linux系统，需要为android sdk tool安装一些32位的类库

        sudo apt-get install libstdc++6:i386 libgcc1:i386 zlib1g:i386 libncurses5:i386
        sudo apt-get install libsdl1.2debian:i386

* 如果发现打包的app无法打开任何url，说明忘了装插件了，在项目目录下执行

        cordova plugin add https://github.com/apache/cordova-plugin-whitelist.git

* 遇到nodejs版本过低，执行下列命令更新

        sudo add-apt-repository ppa:chris-lea/node.js
        sudo apt-get update
        sudo apt-get install nodejs

* 如果列表有很多条，会导致app性能很差，使用colletcion-repeat代替

### 打包app
    
在项目目录下执行

    ionic platform add android
    ionic build android

### 集成crosswalk， 让app性能提升几倍      

在项目目录下执行，如果添加不上，使用root执行

    ionic browser add crosswalk 


### 总结
    
ionic 社区为实现的功能提供了完善的文档，使得开发者很容易入门，建议在过了一把瘾之后把文档好好看看，可以让你开发的更得心应手。

如果您想了解一下有哪些应用是使用ionic做的，并且想下载体验，使用下列链接从googleplay下载ionic官网展示的应用，你会大吃一惊。

http://apps.evozi.com/apk-downloader/
