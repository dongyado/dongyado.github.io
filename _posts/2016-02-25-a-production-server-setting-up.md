---
layout: post
title: 记一次生产服务器基本配置过程
date: 2016-02-25 
categories:
- linux
tags: [linux]
status: publish
type: post
published: true
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
  first_name: ''
  last_name: ''
---
记一次服务器配置，方便以后配置的时候参考。
用的是阿里云的，ubuntu 14.04 64位。

### 分区 - fdisk

~~~ 
root@iZ94u987x7zZ:/# fdisk /dev/xvdb // 需要分区的磁盘
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0xe55a407e.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.

Command (m for help): n
Partition type:
p primary (0 primary, 0 extended, 4 free)
e extended
Select (default p): p // 选择主分区

Partition number (1-4, default 1): 1
First sector (2048-419430399, default 2048): 
Using default value 2048 
Last sector, +sectors or +size{K,M,G} (2048-419430399, default 419430399): // 全部分到一个分区
Using default value 419430399

Command (m for help): wq // 写入并退出
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.

root@iZ94u987x7zZ:/# fdisk -l // 重新列出磁盘信息

Disk /dev/xvda: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders, total 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000da36d

Device Boot Start End Blocks Id System
/dev/xvda1 * 2048 41940991 20969472 83 Linux 

Disk /dev/xvdb: 214.7 GB, 214748364800 bytes // 这里是分好的分区
86 heads, 25 sectors/track, 195083 cylinders, total 419430400 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xe55a407e

Device Boot Start End Blocks Id System
/dev/xvdb1 2048 419430399 209714176 83 Linux
    
~~~
### 格式化磁盘

    root@iZ94u987x7zZ:/data0# mkfs -t ext4 /dev/xvdb1 
    mke2fs 1.42.9 (4-Feb-2014)
    Filesystem label=
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    13107200 inodes, 52428544 blocks
    2621427 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=4294967296
    1600 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks: 
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
    4096000, 7962624, 11239424, 20480000, 23887872
    
    Allocating group tables: done 
    Writing inode tables: done 
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done
    
### 开机挂载磁盘
* 查看磁盘uuid
    
        root@iZ94u987x7zZ:/data0# blkid 
        /dev/xvda1: UUID="af414ad8-9936-46cd-b074-528854656fcd" TYPE="ext4" 
        /dev/xvdb1: UUID="ec60cab1-05ba-4afb-8428-056975c6934e" TYPE="ext4" 

* 添加到fstab挂载到指定目录
vim /etc/fstab 加入：

        UUID=ec60cab1-05ba-4afb-8428-056975c6934e /data0 ext4 defaults 0 1

### 支持中文 
    apt-get install language-pack-zh-hans

### 创建普通用户
* 添加用户reach，并加入到adm和sudo组：

        useradd -d /home/reach -G adm,sudo reach -s /bin/bash
* 初始化reach用户的密码：
    
        passwd "reach"

* 初始化用户目录文件
        
        cp -r "/etc/skel" "/home/reach"
* 需要注意的地方
    如果需要将一个用户添加到用户组中，千万不能直接用： 

        usermod -G groupA 


    这样做会使你离开其他用户组，仅仅做为 这个用户组 groupA 的成员。 
    应该用加上 -a 选项： 

        usermod -a -G sudo reach

    -a 代表 append， 也就是 将reach添加到 用户组sudo中，但是没有离开其他用户组。

### 禁用root远程登录
配好普通用户并加入sudo组后，就可以禁用root用户远程登录了
生产机器禁止ROOT远程SSH登录：

    vim /etc/ssh/sshd_config
把

    PermitRootLogin yes
改为

    PermitRootLogin no
重启sshd服务

    service sshd restart

基本配置完成了，然后就可以安装nginx,mysql,php5等其他主要东西了，再配置iptables，内核参数，就可以重启测试了
