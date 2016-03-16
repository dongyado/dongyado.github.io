---
layout: post
title: zabbix使用外部smtp服务器发送告警邮件
date: 2016-03-15
categories:
- linux
- zabbix
tags: [zabbix,heirloom-mailx]
status: publish
type: post
published: true
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---
使用sendmail发送报警邮件很麻烦，在服务器上装一个sendmail完全没有必要，所以采用外部smtp邮件服务器来发送报警邮件。
试过QQ邮箱，应该是做了限制，无法发送成功，然后再试了163邮箱，能够正常发送。

### 安装heirloom-mailx

	sudo apt-get install heirloom-mailx

### 编辑nail.rc配置
sudo nano /etc/nail.rc 添加网易163邮箱开放的需要认证的smtp服务器: 

	set from=USER@163.com
	set smtp=smtp.163.com
	set smtp-auth-user=USER
	set smtp-auth-password=PASSWORD
	set smtp-auth=login

### 发送脚本 mail.sh

	#!/bin/bash

	to=$1
	subject=$2
	body=$3

	cat <<EOF | heirloom-mailx -s "$subject" "$to"
	$body
	EOF

### 配置zabbix的媒体类型：
To configure custom alertscripts as the media type:

Go to Administration→Media types

Click on Create media typetype: script

script name: main.sh 

参数填三个

	{ALERT.SENDTO}
	{ALERT.SUBJECT}
	{ALERT.MESSAGE}


### 给用户绑定媒体
Go to Administration→Users 

Click on Admin or other user -> media -> click Add to add a media for the user 


Type： 选 email
send to： 填 the email of the user 


### 配置action
Configuration->Actions 
创建需要的action

### 关于邮件正文变成附件以及中文乱码的问题
经查是因为邮件正文包含\r\n，导致邮件出错，所以需要先将正文使用dos2unix处理一下：

	#!/bin/sh
	# 解决中文乱码问题
	export LANG=zh_CN.UTF-8
	to=$1
	subject=$2
	body=$3

	# 方法1, 创建一个临时文件
	FILE=/tmp/mailtmp.txt
	if [ ! -f $FILE ];then 
	touch $FILE
	chown zabbix.zabbix $FILE
	fi

	echo "$body" > $FILE
	dos2unix -k $FILE
	heirloom-mailx -s "$subject" $to <$FILE

	# 方法2, 管道输出
	echo "$body" | dos2unix -k | heirloom-mailx -s "$subject" "$to"

处理完最后两个问题后，系统邮件发送已经正常，就是发送太频繁的时候，偶尔会被QQ邮箱拦截，不过完全可以满足日常监控了。



