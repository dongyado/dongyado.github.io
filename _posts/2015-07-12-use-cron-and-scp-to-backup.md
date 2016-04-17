---
layout: post
title: 使用cron和scp备份数据(ubuntu)
date: 2015-07-13 23:33:18.000000000 +08:00
categories:
- linux
- tool 
tags: [cron, scp]
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  views: '1'
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
  first_name: ''
  last_name: ''
description: '是一个很好的选择，但是也有几个坑需要注意一下'
---

<!-- more -->
### 使用cron和scp备份数据

数据多备份几份，永远也不会错。如果数据量小，又没有条件买专用的备份服务器，本地的电脑也可以作为备份机子，一则数据多了一层保障，二则备份下来的数据，也可以更新本地测试环境的数据，一举两得。

### 为啥选cron和scp
平时在服务器间拷数据用的最多的就是scp，用起来还是很顺手的；本地电脑只能在半夜备份（不影响办公），cron自然就是首选了。

### 思路
先写一个scp脚本，找出服务器上最近的数据备份版本，使用cron定时在凌晨执行脚本备份。
#### scp脚本
*  配置文件脚本，用来配置服务器相关信息
	
		# config file for local backup 
		# server config
		ServerIp=127.0.0.1
		ServerUser=user
		ServerPwd=pass
		ServerBackupHour=( "22"  "14" "05" )
		Path=/data1/backup/
	
*  备份脚本
	服务器会按固定的三个时间点以小时为版本备份数据，脚本只要找出最近的版本备份即可。
	scp输入密码可以使用key解决，这里采用了sshpass传递。
	脚本手动执行，运行很正常。

        {% highlight bash  %}
		#!/bin/bash

		# include config file
		source /etc/local_backup_config.sh 

		# path
		BackupPath=/home/slayer/databackup/
		LogPath=/tmp/local_backup.log

		# date
		BackupDate=$(date +%Y_%m_%d)
		

		# find the latest backup version to backup
		Hour=`date +%H`

		BackupDir=${BackupDate}_${Hour}
		for hour in ${ServerBackupHour[@]}; do
		    if [ "${BackupDate}_${Hour}" \> "${BackupDate}_${hour}" ]
		    then
		       BackupDir="${BackupDate}_${hour}";
		       break;
		    fi
		done

		echo `date +%Y-%m-%d %H:%I:%S`  backup ${BackupDir} ... >> $LogPath

		# backup
		# sshpass -p ${ServerPwd} scp -rv ${ServerUser}@${ServerIp}:${ServerPath}${BackupDir} $BackupPath 1>$LogPath 2>&1 

		echo `date +%Y-%m-%d %H:%I:%S` excute code $? >>$LogPath

		if [ $? -eq 0 ]
		then
		    echo `date +%Y-%m-%d %H:%I:%S` backup ${BackupDir} success >> $LogPath
		else 
		    echo `date +%Y-%m-%d %H:%I:%S` backup ${BackupDir} failed >> $LogPath
		fi
		
        {%  endhighlight %}

#### cron任务
local_backup_cron任务，放入/etc/cron.d/，在凌晨1点05分执行脚本：
	
	# 5 1 * * *  root /home/slayer/vhost/sysadmin/shells/local_backup.sh >/dev/null

这里遇到一个很大的坑，cron到时间确实会执行脚本，但是scp却没有运行成功，查了很多资料，大部分都是说cron的环境变量与当前手动执行的环境不一样，最明显的就是PATH不一样，所以当前命令行能运行的命令，在cron的运行环境里面可能就找不到。
然后就一步一步排查：
* 我试着打印出来cron的PATH，发现其值为 /usr/bin:/bin；但是脚本用到的sshpass，scp都在/usr/bin下面，说明是没有问题的。

* 然后打开了scp的debug模式，在log下发现scp会无缘无故断掉，跑到服务器上看下，服务器显示的信息就是客户端关闭了链接，那就是客户端的scp链接有问题，突然想起来是定时脚本是使用root脚本，顿时想起ssh链接的时候需要去用户目录下.ssh验证一下一些信息，而我从没有在root下链接过那台服务器，恩，改成当前用户就可以了。

		# 5 1 * * *  slayer /home/slayer/vhost/sysadmin/shells/local_backup.sh >/dev/null

#### 加上自动关机
为了省电，在数据下载完成后需要关闭计算机，那就必须使用root用户了，恩没办法了，只能在root下连了一下服务器；

但是shutdown命令并没有在/usr/bin和/bin里面，而是在/sbin里面，那只能在脚本最前面把/sbin加入到PATH：

	PATH=$PATH:/sbin

在备份脚本里面加上关机的代码：

	# shutdown system after job done
	Halt=true
	# shutdown system
	if $Halt ; then
	    echo `date +%Y-%m-%d %H:%I:%S` shutdown... >> $LogPath
	    shutdown -h now >> $LogPath
	    echo `date +%Y-%m-%d %H:%I:%S` shutdown $? >> $LogPath
	fi

### cron里面的坑

因为cron的环境变量不一样的坑，两种解决办法：
* 脚本里面的命令使用绝对路径
* 在脚本里面重新设置PATH

最后使用ssh目录的时候，不管是使用sshpass传递，还是sshkey登录，都需要注意cron任务里面的用户目录下的.ssh目录下存在目标计算机的链接信息。如果该用户没有链接过目标计算机，链接一下，就能跳出这个坑。



