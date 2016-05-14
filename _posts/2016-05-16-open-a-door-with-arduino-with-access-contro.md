---
layout: post
title: 使用 arduino + 树莓派 实现远程控制开门 — 为API加上权限验证
date: 2016-05-14
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
上一节已经完成了web api调用控制arduino了，但是API还没有加上权限验证，API可以直接被外网访问肯定是不安全的，所以有必要加上一层授权和权限验证。

### 流程

访问index.php -> 验证 access_token -> 

- 通过 -> 展示开门操作界面

- 不通过 -> 登录界面 -> 登录 -> 生成access_token -> 跳转 -> index.php?access_token=

### 生成access_token
把用户名，密码，salt串起来加密：

~~~php
<?php
    public static function generateToken($conf){
        return md5("!".$conf['user']."@". $conf['password'] ."#".$conf['salt']);
    }
?>
~~~
觉得不够复杂的，可以继续添加各种字符或者使用sha系列加密函数。

$conf数组被定义在一个配置文件中 config.php:

~~~php
<?php

return array(
    'user'      => 'username',
    'password'  => 'password',
    'salt'      => 'some strings',
    'email'     => 'email', // email to notify when ip changed

    'router_host'   => '192.168.1.1',
    'router_user'   => 'user',
    'router_passwd' => 'password' 
);

?>
~~~

### 验证和登录

验证是否登录，已登录直接生成access_token并跳转， 未登录跳转到登录页面： 

~~~php
<?php
require "./tools/Util.php";
$conf = include "config.php";
// access check
$auth = false;
$auth_code = "";
if (isset($_GET['access_token']))
{
    $access_token = trim($_GET['access_token']);
    if (strlen($access_token) == 32) {
        $sign = Util::generateToken($conf);
        if (strncmp($access_token, $sign, 32) == 0) {
            $auth = true;
            $auth_code = $sign;
        }
    } 
}
// handle action
if (isset($_GET['action'])) {
    $action = trim($_GET['action']);
    if ($auth) {
        // open the door
        if ($action == 'open')
        {
            /**
             * visit operation will cause the arduino work inconrrect  ,
             * but work fine if in cli mode 
             * */
            exec("php operation.php {$action}");
            exit("Opened");
        }
    } else {
        if ($action == 'login') {
            
            $user     = isset($_POST['user']) ? trim($_POST['user']) : ""; 
            $password = isset($_POST['password']) ? trim($_POST['password']) : ""; 
            if ($user== $conf['user'] && $password == $conf['password'])
            {
                $location = "index.php?access_token=".Util::generateToken($conf);
                header("Location:{$location}");
            }
        }
    }
    exit("Failed");
}
?>


<?php 
// 根据授权参数分别展示操作界面和登录界面
if ($auth) {
?>
<a class="button" onclick="ArduinoCtrl.open(this);" >Open</a>
<?php    
} else {
?>
<div class="form" >
<form action="?action=login" method="post" >
<input class="text" type="text" name="user" />
<input class="text" type="password" name="password" />
<input class="text" type="submit" value="登录" />
</div>
</form>
<?php
}
?>
~~~



详细代码见： [index.php][]

该主题其他文章： 
[Remote-control-door][]

相关链接：

- [remote-open-door][] - 项目源码
- [index.php][] - API入口

[remote-open-door]: https://github.com/dongyado/remote-open-door
[index.php]: https://github.com/dongyado/remote-open-door/blob/master/index.php
[Remote-control-door]: http://dongyado.com/categories/#remote-control-door-ref
