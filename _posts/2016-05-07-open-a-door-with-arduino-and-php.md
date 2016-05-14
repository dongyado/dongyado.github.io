---
layout: post
title: 使用 arduino + 树莓派 实现远程控制开门 — 调用 php API控制arduino
date: 2016-05-07
categories:
- remote-control-door
- tool
- funny
tags: [funny,toss,arduino]
status: publish
type: post
published: true
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---

上一节已经把servo的控制程序写好了，可以通过arduino IDE 中的串口监视器给arduino发送指令来控制开门。

这一节介绍如何远程调用API控制arduino开门。

### 流程

客户端 -> 调用API -> 发送指令 -> arduino -> 开门

api调用请求经过下面几个步骤：

手机 -> 发送请求 -> 路由器 -> 转发给内网web服务器 -> 服务器执行api，发送指令

### 硬件环境
客户端 
路由器 -> 内网机器 -> arduino -> 舵机

### 实现
- 搭建好内网web服务器
  不差钱请用台式机，差钱的用闲置的笔记本，付不起电费的，树莓派能拯救你。
  
  搭建好web服务, 这里使用的是ubuntu + nginx + php

- 设置虚拟服务器，在路由器设置端口转发，转发某个端口的请求到内网机器

- 实现php发送指令给arduino

前一节已经介绍了把arduino链接到电脑上，链接后会在/dev目录下产生一个设备文件， 在ubuntu下是/dev/ttyACM0， 系统与arduino的通信都是通过这个文件来实现的，
所以也可以使用php给这个设备发送指令。

之前提过arduino是可以通过串口通信，github上已经有这个实现： [PhpSerail][]

这个类做了两个动作：

  - 通过stty 命令初始化/dev/ttyACM0 为串口设备
  - 使用fwrite向设备发送数据

为了快速实现调用，我毫不犹豫的使用了这个轮子，具体调用代码如下：

~~~php

<?php
/**
 * operation file
 *
 * communication with arduino by /dev/tty
 *
 * */
include 'PhpSerial.php';
$serial = new PhpSerial;

if (php_sapi_name() == "cli") {
    $action = trim( $argv[1] );  // 获取参数，可能为open或者1
} else {
    exit("Access Denied");
}


// First we must specify the device. This works on both linux and windows (if
// your linux serial device is /dev/ttyS0 for COM1, etc)

$serial->deviceSet("/dev/ttyACM0"); // 这里设置设备路径

// We can change the baud rate, parity, length, stop bits, flow control
$serial->confBaudRate(9600); // 设置波特率，跟servo程序里面的 Serial.begin(9600); 相对应
$serial->confParity("none");
$serial->confCharacterLength(8);
$serial->confStopBits(2);
$serial->confFlowControl("none");

// Then we need to open it
$serial->deviceOpen("w+");

// must sleep at least 2 seconds
// 这里必须延迟至少2秒来保证数据能够被arduino接收
sleep(2);

// To write into
$serial->sendMessage($action, 1);

// Or to read from
$read = $serial->readPort(9600);
//echo $read."\n===\n";
// If you want to change the configuration, the device must be closed
$serial->deviceClose();

?>
~~~

如果直接web调用这个文件，arduino偶尔会出现重复操作（开门两次），但是在cli模式下不会出现这个问题，这是个很奇怪的问题，还没有查出具体原因，所以先想了简单的办法解决。

在index.php执行operation.php:

~~~php
<?php
    exec("php operation.php {$action}");
?>
~~~

全部代码：

~~~php
<?php

if (isset($_GET['action'])) {
  
    $action = trim($_GET['action']);
    // open the door
    if ($action == 'open') //在servo的控制程序使用了open作为开门的指令
    {
        /**
          * visit operation will cause the arduino work inconrrect  ,
          * but work fine if in cli mode 
          * */
        exec("php operation.php {$action}");  // 执行operation.php 
        exit("Opened");
    }

    exit("Failed");
}
?>
~~~

现在已经可以直接访问 http://host/index.php?action=open 控制开门了，只要能访问到家里的路由器，就能随时执行开门的动作，awesome!


下一节包括以下内容：

- 为api加上权限验证
- 检测路由器IP变化并发送邮件通知的脚本

该主题其他文章： 
[Remote-control-door][]

相关链接：

- [remote-open-door][] - 项目源码
- [servo][] - 舵机(servo)控制程序
- [PhpSerail][] 

[PhpSerail]: https://github.com/dongyado/remote-open-door/blob/master/PhpSerial.php
[remote-open-door]: https://github.com/dongyado/remote-open-door
[servo]: https://github.com/dongyado/remote-open-door/blob/master/arduino_control/servo/servo.ino
[Remote-control-door]: http://dongyado.com/categories/#remote-control-door-ref

