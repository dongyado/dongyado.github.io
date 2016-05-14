---
layout: post
title: 使用 arduino + 树莓派 实现远程控制开门 — arduino控制舵机
date: 2016-04-27
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

arduino就是一块神奇的板子，为嘛神奇，因为外行也能用它作出不可思议的东西。
下面先介绍一下如何使用arduino控制舵机，最终实现接电话开门的动作。


### 材料

* 板子，某宝可入正版，90大洋能搞定。
* 舵机，10大洋以内可入。长这样,上面装了个桨。
    
 ![servo](/images/post/servo1.jpg)   

### 入门

- 先把板子和电脑用自带的USB连上，因为驱动的东西不多，就直接使用USB供电了。
- 下载安装arduino IDE, windows,mac,linux版本都有，只需在官网下载即可使用。
- IDE提供了很多示例程序，直接拿过来使用，稍微修改一下就能直接上传到板子使用，很方便。
- IDE也提供了串口通讯监视器，可直接发送数据给arduino，也可以接收arduino输出的数据，为调试提供了很大的便利。
- 要注意个地方在于，linux下的权限问题， 串口通讯需要往串口输入数据的权限，采用下面的命令修改权限

      sudo usermod -a -G dialout username 
      sudo chmod a+rw /dev/ttyACM0

  /dev/ttyACM0 就是arduino设备

  其他的资料基本可以自行脑补，就不多敷述了。

### 连接线路
普通舵机有三根线：GND（黑，接地）、VCC（红，电源）、Signal（黄，信号）
接线如下：

![servo](/images/post/arduino-servo.png)

GND表示ground的意思，接地； vcc 接5v电源； signal 接9号针脚。


### 测试
arduino IDE自带了很多示例,选一个舵机的示例测试一下,打开
arduino ide -> 文件 -> 示例 -> 第三方示例 -> Servo -> sweep
源码如下：

~~~c
/* Sweep
 by BARRAGAN <http://barraganstudio.com>
 This example code is in the public domain.

 modified 8 Nov 2013
 by Scott Fitzgerald
 http://www.arduino.cc/en/Tutorial/Sweep
*/

#include <Servo.h>

// 创建一个servo(舵机)控制对象
Servo myservo;  // create servo object to control a servo
// twelve servo objects can be created on most boards

// 储存舵机位置
int pos = 0;    // variable to store the servo position

// 初始化
void setup() {
  // 绑定myservo到9号针脚,后面只能通过9号针脚控制myservo
  myservo.attach(9);  // attaches the servo on pin 9 to the servo object
}

// 死循环
void loop() {
  // 从0转到180度
  for (pos = 0; pos <= 180; pos += 1) { // goes from 0 degrees to 180 degrees
    // in steps of 1 degree
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                       // waits 15ms for the servo to reach the position
  }
  
  // 从180度转到0度
  for (pos = 180; pos >= 0; pos -= 1) { // goes from 180 degrees to 0 degrees
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                       // waits 15ms for the servo to reach the position
  }
}
~~~

如上，arduino 控制程序主要只有两个方法，setup和loop。
setup初始化一些东西, 比如绑定舵机到9号针脚。
在loop里面实现控制程序。 

arduino从一启动先执行setup,然后进入loop死循环,所以loop才是主战场。

### 控制两个舵机,实现开门
接电话分三个动作: 

* 接 - 拿起电话,放电话的槽中有个按钮会弹起来,从而接通
* 按 - 右侧边有个开门的按钮, 按下去,会把门打开
* 挂 - 把电话挂掉,把第一步的那个按钮按下去

如下图摆放(做的很粗糙,拍照技术也渣),

左側phoneServo否则接电话和挂电话, 右側unlockServo负责开门。 
phoneServo需要先初始化为90度,把按钮按下去; unlockServo初始化为0度。

![servo](/images/post/servo3.jpg)

流程清楚了,程序也很简单,顺便加了串口通讯在里面。如下:

~~~c
/**
 * servo control program
 * 
 * @author dongyado<dongyado@gmail.com>
 */

#include<Servo.h>

Servo unlockServo; // 开门舵机
Servo phoneServo;  // 电话舵机

int pos = 0;

int maxPhonePos  = 90; // phoneServe最大角度
int maxUnlockPos = 30; // unlockServe最大角度

String comdata = ""; // 存储接收的指令

void setup() {
  
    Serial.begin(9600); // 监听串口,波特率9600
    
    // connect 
    unlockServo.attach(9);
    phoneServo.attach(10);
  
    //reset
    unlockServo.write(0);
    phoneServo.write(maxPhonePos);
    
    delay(200); // 延时是为了等待舵机转动完成
    
    // disconnect
    unlockServo.detach();
    phoneServo.detach();
}

void loop() {

    //接收串口发来的数据
    while (Serial.available() > 0)  
    {
        comdata += char(Serial.read());
        delay(10);
    }
    
    // 接收到数据
    if (comdata.length() > 0)
    {
        // connect 
        unlockServo.attach(9);
        phoneServo.attach(10);

        // 判断指令
        if (comdata == "open" || comdata == "1" ) {
            
            // get the phone up 
            for (pos = maxPhonePos; pos >= 1; pos -=1 ){
              phoneServo.write(pos);
              delay(10);  
            }

            delay(500);
            
            // press the button of open door
            for (pos = 1; pos < maxUnlockPos; pos+=1) {
                unlockServo.write(pos);
                delay(10);
            }

            delay(200);

            //release the unlock button
            for (pos = maxUnlockPos; pos >= 1; pos -=1 ){
              unlockServo.write(pos);
              delay(10);  
            }

            // get the phone down            
            for (pos = 1; pos < maxPhonePos; pos+=1) {
                phoneServo.write(pos);
                delay(10);
            }
        }

        //Serial.println(comdata);

        // disconnect
        unlockServo.detach();
        phoneServo.detach();
    } 

 
    delay(10);
    comdata = "";
}

~~~

以上逻辑很简单,还加了串口通讯,使用自带的串口监视器即可发送open或者1给arduino,到此,arduino接电话开门的动作完成,下一步要做的就是实现远程调用接口开门.


该主题其他文章： 
[Remote-control-door][]

相关链接：

- [remote-open-door][] - 项目源码
- [servo][] - 舵机(servo)控制程序

[remote-open-door]: https://github.com/dongyado/remote-open-door
[servo]: https://github.com/dongyado/remote-open-door/blob/master/arduino_control/servo/servo.ino
[Remote-control-door]: http://dongyado.com/categories/#remote-control-door-ref
