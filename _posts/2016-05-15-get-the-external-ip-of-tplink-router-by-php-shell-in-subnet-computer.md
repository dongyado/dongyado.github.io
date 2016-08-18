---
layout: post
title: 内网机器通过脚本获取tplink路由器的外网IP
date: 2016-05-15
categories:
- remote-control-door
- tool
- funny
tags: [funny,toss]
status: publish
type: post
published: true
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---

一般来说，子网机器是无法直接拿到路由器的IP，
就比如笔记本没法直接获取所连接路由器的IP，只能登录路由器控制台查看到分配给路由器的IP。

之前有一篇博客总结了几种内网机器获取路由器外网IP的方法，查看[several-ways-to-get-router-external-ip][]。

这里实现第三种方法，使用脚本去获取。

### 准备工作
要用脚本模拟登录控制面板，要先弄清楚，浏览器中控制面板是如何登录并获取数据的。

本文使用的路由器： TPL-LINK TK-WR842N， 软件版本：2.3.8 Build 150425 Rel.60768n

登录该路由器的控制面板后，用chrome的开发者工具调试，发现登录和获取数据操作的request header里面没有带任何cookies，
那就只能是通过类似access_token的机制来验证访问权限了。

通过断点调试发现，每隔1秒就有请求发出：

~~~
http:192.168.1.1/?code=2&asyn=1&id=R0TPpo5pDg%3CJr)5k
~~~

request header 里面还传了一个request payload, 数据为： 23， 后面发现这个必须带，可能是一个指令。

请求成功（200），返回的数据类似：

data1:

~~~
  00000 id 23 ip 219.133.xxx.xxx mask 255.255.255.255 gateway 219.133.xxx.xxx dns 0 202.96.128.86 dns 1 202.96.134.133 status 1 code 0 upTime 0 inPkts 2147576644 inOctets 2 outPkts 2149255756 outOctets 0
~~~

很明显，url后面带了一个id, 而且这个id每隔一段时间就会变，也就是说这个id会过期。

继续调试后发现每隔一段时间，会有一个请求失败，返回401（Unauthorized）,返回的数据类似： 

data2:

~~~
00007 00003 00001 }2C2H]7Peba4sBvw ^O09j|cvO)4Eu*$~6KWnOa,c[+$5PIDi3(Ca!ni+VsKgT(N$7ZiR(DIJVp(wt5Up]0vbJ$G2kVq>46b.efn9(tTcx~Kg6swDW61qwt912h|]26cJ0(0+GgWvAh}!.]X[K.4+lDl~!UEd9PfR9vq1~ppk*komxr
~~~

那么很明显没用的数据完全没必要返回，认证失败返回这些信息，而且在下一次请求的时候id变了，也就是说完成了自动认证，那就说js里面肯定有个生成id的函数。

既然已经确定js中肯定有加密函数，那完全可以从登录开始，一步一步调试，经过了几个小时的调试，终于把基本的函数和流程确定下来：

#### 登录操作：

点击登录按钮会直接调用这个函数：

~~~javascript
    function lgDoSub()
    {
        var lgPwd = id("lgPwd"), sessionValue = "";
        var value = lgPwd.value, result, pos, errorCode;

        /* 检查密码 */
        if (value.length > 15 || value.length < 6)
        {
            showLgError(HTTP_CLIENT_NORMAL);
            return;
        }

        if (!lgChkPswVal(value))
        {
            showLgError(HTTP_CLIENT_PSWIlegal);
            return;
        }

        /* 发送密码数据 */
        result = $.auth($.orgAuthPwd(value)); // 这里调用auth函数进行认证

        /* 处理返回的结果 */
        if(result.errorno == ENONE)
        {
            unloadLogin();
            lgPwd.value = "";
        }
        else
        {
            showLgError(parseInt(authInfo[1]));
        }
    }

~~~

根据原始密码生成一个加密后的密码

~~~javascript
	this.orgAuthPwd = function(pwd)
	{
		var strDe = "RDpbLfCPsJZ7fiv";
		var dic = "yLwVl0zKqws7LgKPRQ84Mdt708T1qQ3Ha7xv3H7NyU84p21BriUWBU43odz3iP4rBL3cD02KZciX"+
				  "TysVXiV8ngg6vL48rPJyAUw0HurW20xqxv9aYb4M9wK1Ae0wlro510qXeU07kV57fQMc8L6aLgML"+
				  "wygtc0F10a0Dg70TOoouyFhdysuRMO51yY5ZlOZZLEal1h0t9YQW0Ko7oBwmCAHoic4HYbUyVeU3"+
				  "sfQ1xtXcPcf1aT303wAQhv66qzW";

		return this.securityEncode(pwd, strDe, dic); // pwd为登录时候输入的密码
	};
~~~

认证时会调用这个函数，进行认证操作：

~~~javascript
/* 认证请求 */
	this.auth = function(data)
	{
		var pwd = data;
		var url = this.domainUrl + "?code="+ TDDP_AUTH +"&asyn=0"; //这里生成请求路径

		/* 密码格式错误 */
		if (data == undefined || 0 == pwd.length)
		{
			this.result.errorno = EUNAUTH;
			return this.result;
		}

		data = undefined;
		this.initResult();
    
    /**
     * authInfo是个全局变量，每次返回401都会用下文提到的parseAuthRlt,重新解析获取数据。
     * this.sessiion 对应请求url中的id
     */
		this.session = this.securityEncode(authInfo[3], pwd, authInfo[4]); 
    
		url += ("&id=" + this.encodePara(this.session));

		if ((false == this.local) || (this.routerAlive))
		{
			this.externLoading(true);
			this.request(url, data, "post", this.ajaxSyn); // 发送请求
			this.externLoading(false);
		}

		/* 解析数据 */
		this.parseAuthRlt();

		if (ENONE == this.result.errorno)
		{
			this.setLgPwd(pwd);
		}

		return this.result;
	};
~~~


#### 认证数据解析函数

用来解析上文提到的data2:

~~~
00007 
00003 
00001 
2xDsMK3]2DM4b(]J 
L+JKFtNljcWoysK$aKf4x6Ud32kvWAWT++i)r+ITeF!U]k}u0+LM3MVd)mIn}p2zMtxi<7Wy9Vx7!x0k>GE{WMtu>a!k^7PCW+si!Ime9OvH{7Z.A0P]$LC4jF(WplrI[>0[kaBBvRPBP4GIV*au!3xa0N1Y7U^$Wgc(.xyf}BAG(Ev4Ei1sK+c{SuOpCG0^W8of}B0$+JK[oYys2]$ZXOhkb.bn!L$$>4pErmg*[kWYAgqP]sn44Fm3j
~~~

最后两行分别被存储到authInfo[3]和authInfo[4]，上文this.auth函数中用到了这两个数据。

~~~javascript
this.parseAuthRlt = function()
{
    var results;
    var relCnt = this.result.data;

    if (relCnt.lastIndexOf("\r\n") == relCnt.length - 2)
    {
        relCnt = relCnt.substring(0, relCnt.length - 2);
    }

    results = relCnt.split("\r\n");

    if (EUNAUTH == this.result.errorno)
    {
        authInfo[1] = results[0];
        authInfo[2] = results[1];
        authInfo[3] = results[2]; // 生成id的掩码
        authInfo[4] = results[3]; // 生成id的字典
        $.group = results[4];
        $.pagePRHandle();
    }

    return results;
};

~~~

#### 加密函数：

~~~javascript
this.securityEncode = function(input1, input2, input3)
{
  var dictionary = input3;
  var output = "";
  var len, len1, len2, lenDict;
  var cl = 0xBB, cr = 0xBB;

  len1 = input1.length;
  len2 = input2.length;
  lenDict = dictionary.length;
  len = len1 > len2 ? len1 : len2;

  for (var index = 0; index < len; index++)
  {
    cl = 0xBB;
    cr = 0xBB;

    if (index >= len1)
    {
      cr = input2.charCodeAt(index);
    }
    else if (index >= len2)
    {
      cl = input1.charCodeAt(index);
    }
    else
    {
      cl = input1.charCodeAt(index);
      cr = input2.charCodeAt(index);
    }

    output += dictionary.charAt((cl ^ cr)%lenDict);
  }

  return output;
};
~~~

### 确定流程

现在可以确定流程如下：

登录流程：

~~~
输入密码->提交-> 加密原始密码并认证 $.auth($.orgAuthPwd(value)); -> 解析返回数据
~~~

数据获取流程：

~~~
获取数据请求 -> 401 -> 解析数据到authInfo -> 重新生成id -> 利用新id继续发送请求 -> 返回正常数据
~~~

根据流程和以上提到的加密函数，就可以写出一个脚本：

~~~php
<?php
/**
 * tplink router control 
 * 
 * simulate login to get  external ip, 
 * and send current ip to someone by email when the ip changed 
 *
 * @author dongyado<dongyado@gmail.com>
 */
require "./tools/Util.php";
require("./tools/HttpClient.class.php");
$conf = include "config.php";


/**
* encrypt function
*/
function securityEncode($input1, $input2, $input3) {
    $dictionary = $input3;
    $output  = "";
    $cl = 0xBB;
    $cr = 0xBB;
    
    $len1 = strlen($input1);
    $len2 = strlen($input2);
    $lenDict = strlen($dictionary);
    
    $len = $len1 > $len2 ? $len1 : $len2;
    for( $index = 0; $index < $len; $index++){
        $cl = 0xBB;
        $cr = 0xBB;
        
        if ($index >= $len1) {
            $cr = ord($input2[$index]);
        }
        else if ($index >= $len2) {
            $cl = ord($input1[$index]);
        }else {
            $cl = ord($input1[$index]);
            $cr = ord($input2[$index]);
        }
        
        $tmp = ($cl ^ $cr) % $lenDict;
        $output .= $dictionary[$tmp];
    }    
    return $output;
}


/**
* password encrypt from original password
* 
*/
function orgAuthPwd($pwd) {
    	$strDe = "RDpbLfCPsJZ7fiv";
	$dic = "yLwVl0zKqws7LgKPRQ84Mdt708T1qQ3Ha7xv3H7NyU84p21BriUWBU43odz3iP4rBL3cD02KZciX".
	  "TysVXiV8ngg6vL48rPJyAUw0HurW20xqxv9aYb4M9wK1Ae0wlro510qXeU07kV57fQMc8L6aLgML".
	  "wygtc0F10a0Dg70TOoouyFhdysuRMO51yY5ZlOZZLEal1h0t9YQW0Ko7oBwmCAHoic4HYbUyVeU3".
	  "sfQ1xtXcPcf1aT303wAQhv66qzW";
	return securityEncode($pwd, $strDe, $dic);
}


// ecrypted orginal password
$password = orgAuthPwd($conf['router_passwd']);
$httpClient = new HttpClient($conf['router_host']);


// request example
// http:192.168.1.1/?code=2&asyn=1&id=R0TPpo5pDg%3CJr)5k

$status = 401;
$id = ""; // global id to store newest id
$opath = "?code=2&asyn=1"; // prefix of path 
$duration = 0;
$ip = "";


// loop to check the external ip
while(true) {
    
    $path = $id == "" ? $opath : $opath . "&id={$id}";
    $ret = $httpClient->post($path, array(23));
    
    // get the reponse data after login
    echo "status: ".$ret['status']."\n";
    
    $status = $ret['status'];
    // parse body
    $data = preg_split("/\r\n/", $ret['body']);
    
    // response 401, Unauthoried
    if ($status == 401 || $id == "") {
        echo "auth data:" .json_encode($data)."\n";
        // generate new id
        $id = securityEncode($data[3], $password, $data[4]);
        echo "id: {$id}\n";
        //file_put_contents("./log", "[duration] :" . (time() - $duration)."\n", FILE_APPEND);
        sleep(5);
        continue; 
    } 
    
    // response ok		
    // parse data
    $_data = array();
    foreach($data as $item) {
        $record = explode(" ", $item);
        if(count($record) > 1)
            $_data[$record[0]] = $record[1];
        else 
            $_data[] = $item;
    }
    
    
    // send email if necessary
    if ( ($ip == "") || ($ip != "" && $_data['ip'] != $ip)) {
        $token = Util::generateToken($conf);
        // send email
        exec('./tools/mail.sh "'.$conf['email'].'" "ipchanged"  "'.date('Y-m-d H:i:s')." http://".$_data['ip'].':88/?access_token='.$token.'"');        
    }
    $ip = $_data['ip'];
    echo "[".date('Y-m-d H:i:s')."] ip: {$_data['ip']}\n";
    
    // sleep 2 minutes
    sleep(120);
}
?>

~~~

脚本增加了在IP改变时发送邮件通知的功能，使用了mail.sh脚本，也用到了config.php中的配置信息，源码参见：

- [remote-open-door][] - 项目源码
- [getExternalIp.php][] - 本文实现的脚本

[remote-open-door]: https://github.com/dongyado/remote-open-door
[getExternalIp.php]: https://github.com/dongyado/remote-open-door/blob/master/getExternalIp.php
[several-ways-to-get-router-external-ip]:/tool/funny/2016/04/17/several-ways-to-get-router-external-ip/
