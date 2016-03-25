---
layout: post
title: zabbix安装与配置（ubuntu 14.04） 
date: 2016-03-13
categories:
- linux
- zabbix
tags: [zabbix]
status: publish
type: post
published: true
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---
之前只配过nagios，最近觉得有必要试用一下其它的监控平台，然后再选一个合适的放生产环境中使用，然后看到zabbix不错，就适用了一下，发现安装简单，功能也简单易用，瞬起好感，遂把安装过程记下来，以后需要的时候查看。

### 一 服务端
Zabbix 3.0 for Ubuntu 14.04 LTS:

    # wget http://repo.zabbix.com/zabbix/3.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.0-1+trusty_all.deb
    # dpkg -i zabbix-release_3.0-1+trusty_all.deb
    # apt-get update

* 安装服务端（使用mysql做数据存储）和web服务

        # apt-get install zabbix-server-mysql zabbix-frontend-php

	在未安装apache的情况下，会遇到这个错误：

		/var/lib/dpkg/info/zabbix-frontend-php.postinst: 24: /var/lib/dpkg/info/zabbix-frontend-php.postinst: /usr/sbin/a2enconf: not found
		dpkg: error processing package zabbix-frontend-php (--configure):
		 subprocess installed post-installation script returned error exit status 127
		Processing triggers for libc-bin (2.19-0ubuntu6) ...
		Processing triggers for ureadahead (0.100.0-16) ...
		Errors were encountered while processing:
		 zabbix-frontend-php
		E: Sub-process /usr/bin/dpkg returned an error code (1)

	因为缺失 apache的一个配置文件
	/usr/sbin/a2enconf: not found
	zabbix-frontend-php配置失败，因为我使用的是nginx和php5-fpm，下一步照着安装产生的配置文件修改一下即可。


	1). 查看生成的 apache 主机文件

	zabbix-frontend-php安装产生的apache虚拟主机配置文件在/etc/apache2/conf-available里面，然后链接到了/etc/zabbix/apache.conf。
	该配置文件指定了 frontend-php 的web文件在 /usr/share/zabbix

	2). 拷贝web文件

	为了安全和方便管理，最好把 zabbix-frontend-php的web文件（/usr/share/zabbix），拷贝到nginx的web目录下。

	3). 配置主机文件
    
    3.0版本:

    我们需要根据/etc/zabbix/apache.conf, 写一个nginx 的主机文件，
	再拷贝到/etc/nginx/site-available，指定网站根目录，禁止访问一些目录，配置好后， reload nginx就可以使用了。

    2.0版本：
    安装时，已经默认拷贝了nginx的配置文件，一般在：
    /usr/share/doc/zabbix-frontend-php/examples/nginx.conf
    可以根据这个写个虚拟主机文件。
    


* 初始化数据库：

	用终端连接mysql:

	    mysql -uroot -p

	1). 创建数据库：

	    mysql> create database zabbix character set utf8 collate utf8_general_ci; 
	    mysql> insert into mysql.user(Host,User,Password) values('%','zabbix',password('zabbix'));
	    mysql> flush privileges;
	    mysql> grant all privileges on zabbix.* to zabbix@'%' identified by 'zabbix';
	    mysql> flush privileges;

	2). 初始化数据库：

	    # cd /usr/share/doc/zabbix-server-mysql
	    # zcat create.sql.gz | mysql -uroot -p zabbix

	3). 编辑zabbix server 的数据库配置

	    # vi /etc/zabbix/zabbix_server.conf
	    DBHost=localhost
	    DBName=zabbix
	    DBUser=zabbix
	    DBPassword=zabbix

	4). 启动服务

	    # service zabbix-server start

* 配置web服务最低的环境

	按照下面的提示(在/etc/zabbix/apache.conf里面)，修改fpm的php.ini，以下值都是最低要求，可以按需求适度加大
	    
		max_execution_time= 300
		memory_limit= 128M
		post_max_size = 16M
		upload_max_filesize = 2M
		max_input_time = 300
		date.timezone = Asia/Shanghai
	修改完成后，reload php5-fpm即可，然后就可以使用绑定好的域名访问了。

* 打开配置好的web服务域名，按照提示一步一步填写，然后就可以使用Admin和zabbix登录后台了。


### 二 客户端
使用下面的命令安装

    # apt-get install zabbix-agent

编辑 vim /etc/zabbix/zabbix_agentd.conf

    Server=127.0.0.1 # 监控服务器的IP
    ListenPort=10050 # 监听的端口
    StartAgents=1    # 启动的客户端进程
    ServerActive=127.0.0.1：10051 # 主动模式下的监控服务器IP（主动模式必须）
    Hostname=Zabbix server # 监控服务器的名称，大小写敏感（主动模式必须）

以上信息必须正确（省掉了用户密码）， 监控服务器才能从客户端获取到数据。

主动模式 表示客户端提交数据到监控主服务器
被动模式 表示监控主服务器定时去客户端取数据

### 三 结语
安装配置下来，总体来说要比nagios简单易用很多，后面还可以结合salt进行自动化运维，是一个很不错的选择。

