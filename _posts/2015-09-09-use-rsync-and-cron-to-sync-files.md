---
layout: post
title: 使用rsync和cron同步文件
date: 2015-09-09 12:33:18.000000000 +08:00
categories:
- linux
- tool 
tags: [cron, rsync]
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
description: '在linux下使用rsync和cron定时同步文件是一个快速简单的方法'
---

<!-- more -->

### rsync 安装配置

#### 主服务器上配置
ubuntu 默认安装了rsync，但是并没有作为服务启动

##### 基本配置

* 在/etc下创建一个rsync目录 

        $ mkdir -m 600 rsync

* 拷贝配置文件样本

        $ cd rsync/
        
        $ cp /usr/share/doc/rsync/examples/rsyncd.conf ./

* 配置rsyncd.conf

        $:/etc/rsync# vim rsyncd.conf

    修改信息
    
          # sample rsyncd.conf configuration file
          
          # GLOBAL OPTIONS
          
          motd file=/etc/rsync/rsyncd.motd
          log file=/var/log/rsyncd
          # for pid file, do not use /var/run/rsync.pid if
          # you are going to run rsync out of the init.d script.
          # The init.d script does its own pid file handling,
          # so omit the "pid file" line completely in that case.
          pid file=/var/run/rsyncd.pid
          syslog facility=daemon
          #socket options=
          
          # MODULE OPTIONS
          
          [app]
          
              comment = app
              path = /data0/web/app
              use chroot = yes
          #   max connections=10
              lock file = /var/lock/rsyncd
          # the default for read only is yes...
              read only = yes
              list = yes
              uid = nobody
              gid = nogroup
              exclude = uploadfiles/
          #   exclude from = 
          #   include =
          #   include from =
          #   auth users = 
              secrets file = /etc/rsygn/rsyncd.secrets
              strict modes = yes
          #   hosts allow =
          #   hosts deny =
              ignore errors = no
              ignore nonreadable = yes
              transfer logging = no
          #   log format = %t: host %h (%a) %o %f (%l bytes). Total %b bytes.
              timeout = 600
              refuse options = checksum dry-run
              dont compress = *.gz *.tgz *.zip *.z *.rpm *.deb *.iso *.bz2 *.tbz


* 配置 rsyncd.secrets

        $:/etc/rsync# vim rsyncd.secrets
        
    加入用户和密码
        backup:xxxxx

* 配置 rsyncd.motd

        $:/etc/rsync# vim rsyncd.motd
        
    加入
        Welcome Man!
        
##### 配置rsync 服务

    $:/etc/rsync# vim /etc/default/rsync 
    
修改文件, RSYNC_ENABLE 设置为true， 并指定配置文件为/etc/rsync/rsyncd.conf

      # defaults file for rsync daemon mode
      
      # start rsync in daemon mode from init.d script?
      #  only allowed values are "true", "false", and "inetd"
      #  Use "inetd" if you want to start the rsyncd from inetd,
      #  all this does is prevent the init.d script from printing a message
      #  about not starting rsyncd (you still need to modify inetd's config yourself).
      RSYNC_ENABLE=true
      
      # which file should be used as the configuration file for rsync.
      # This file is used instead of the default /etc/rsyncd.conf
      # Warning: This option has no effect if the daemon is accessed
      #          using a remote shell. When using a different file for
      #          rsync you might want to symlink /etc/rsyncd.conf to
      #          that file.
      RSYNC_CONFIG_FILE=/etc/rsync/rsyncd.conf
      
      # what extra options to give rsync --daemon?
      #  that excludes the --daemon; that's always done in the init.d script
      #  Possibilities are:
      #   --address=123.45.67.89      (bind to a specific IP address)
      #   --port=8730             (bind to specified port; default 873)
      RSYNC_OPTS=''
      
      # run rsyncd at a nice level?
      #  the rsync daemon can impact performance due to much I/O and CPU usage,
      #  so you may want to run it at a nicer priority than the default priority.
      #  Allowed values are 0 - 19 inclusive; 10 is a reasonable value.
      RSYNC_NICE=''
      
      # run rsyncd with ionice?
      #  "ionice" does for IO load what "nice" does for CPU load.
      #  As rsync is often used for backups which aren't all that time-critical,
      #  reducing the rsync IO priority will benefit the rest of the system.
      #  See the manpage for ionice for allowed options.
      #  -c3 is recommended, this will run rsync IO at "idle" priority. Uncomment
      #  the next line to activate this.
      # RSYNC_IONICE='-c3'
      
      # Don't forget to create an appropriate config file,
      # else the daemon will not start.

##### 启动服务

    $:/etc/rsync# sudo service rsync start
    * Starting rsync daemon rsync
    
##### 在客户端同步

    rsync -vzrtopg --progress --delete backup@120.24.**.**::app ./


##### 自动同步定时脚本

* shell脚本 vim /data0/shell/rsync.sh
	
	    #!/bin/bash
	    rsync -zrtopg --progress --delete --password-file=/etc/rsync/rsync.secrets backup@120.24.78.68::app /data0/web/app/

* 定时脚本 vim /data0/shell/rsync_cron

        # rsycn crontab shell
        */5 * * * * root sh /data0/shell/rsync.sh >> /data0/log/rsync.log
        
* 加入crontab

        $ crontab /data0/shell/rsync_cron

* 自动加载任务，加到入/etc/cron.d/

	    ln -s /data0/shell/rsync_cron /erc/cron.d/rsync_cron


### 其他
* 开启cron.log, vim /etc/rsyslog.d/50-default.conf，重启rsyslog
* cron每一个任务都是一整行，为防止任务载入失败，所以在每个任务后面都需要打一个换行


