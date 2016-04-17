---
layout: post
title: 栈溢出的原理和利用
date: 2016-03-31
categories:
- linux
- funny
tags: [funny]
status: publish
type: post
published: true
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---

去年偶然看到一本书，讨论了一些常见漏洞，挺有意思的一本书，不是讲找漏洞，而是写漏洞。比起那种千篇一律，教人使用各种工具找漏洞的书，这种书才是从根本出发，也越容易让人接受。

从书中学到了很多，下面结合例子，介绍一下栈溢出漏洞(本文的实验测试全部是在ubuntu 14.04下完成， 不同操作系统可能不一样)。

### 什么是栈溢出
栈溢出，其实就是使用了不该使用的内存，在写入缓冲区的时候，如果忘记检查，就会出现溢出的问题。

### 模拟栈溢出漏洞

看一个C语言的例子，我们知道函数里面的变量会存储在栈里面，我们写一个检查权限的函数：

~~~c

    int check_authentication(char *password)
    {
        int auth_flag = 0;  //存储在栈中
        char password_buffer[12]; //存储在栈中，用来缓冲传进来的password

        strcpy(password_buffer, password); //拷贝password指向的字符串到 password_buffer

        // 检查是否相等
        if (strcmp(password_buffer, "password_text") == 0){
                auth_flag = 1; // 1表示验证通过
        }

        if (strcmp(password_buffer, "stackoverflow") == 0){
                auth_flag = 1;
        }

        printf("P: %s\n", password_buffer);
        return auth_flag;
    }
    
~~~

然后还有一个入口函数：

~~~c

    int main (int argc, char * argv[])
    {
        if (argc < 2)
        {
            printf("Usage: %s <password>\n", argv[0]);
            exit(0);
        }

        //验证通过
        if (check_authentication(argv[1]))
        {
            printf("\n-=-=-=-=-=-=-=-=-=-=-\n");
            printf(" Access Granted.");
            printf("\n-=-=-=-=-=-=-=-=-=-=-\n");
        } else {
            printf("\nAccess Denied!\n");
        }
    }
~~~

把这两段代码加入到一个auth_overflow.c的文件里面，再加上一些必须的头信息：

~~~c

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
~~~

然后我们可以编译运行了:
    
    gcc -g -o auth_overflow.o auth_overflow.c 

运行

    $ ./auth_overflow.o password_text
    password_buffer: password_text
    password_buffer_size: 12
    auth_flag: 1

    -=-=-=-=-=-=-=-=-=-=-
     Access Granted.
    -=-=-=-=-=-=-=-=-=-=-
 

gcc默认编译的时候启用了堆栈保护（[GCC堆栈保护原理][]）。

上面的代码虽然有栈溢出漏洞，由于gcc其实修改了代码，漏洞已经不能被正常利用了。

为了能利用这个这个漏洞，编译的时候加上-fno-stack-protector, 禁用gcc的栈溢出保护优化：

    $ gcc -g -fno-stack-protector -o auth_overflow.o auth_overflow.c 

运行：

    $ ./auth_overflow.o password_text

正常的输入password_text或者stackoverflow测试,会输出 Access Granted

用错误的密码测试， password_buffer的长度是12,首先随便输入12个字符，比如123456789012：

    $ ./auth_overflow.o 123456789012
    password_buffer: 123456789012
    password_buffer_size: 12
    auth_flag: 0

    Access Denied!


结果是不通过，现在把输入的密码长度延长到13，比如 1234567890123：

    $ ./auth_overflow.o 1234567890123
    password_buffer: 1234567890123
    password_buffer_size: 12
    auth_flag: 51

    -=-=-=-=-=-=-=-=-=-=-
     Access Granted.
    -=-=-=-=-=-=-=-=-=-=-

注意看password_buffer已经是13个字符了，为啥size还是12,这个其实就在于C的char数组读取没有检查长度。

更神奇的地方是，只多加了一个3就验证通过了，而且auth_flag竟然是51！ 就是3对应的ANSI编码！
也就是说auth_flag所指向的内存被改写成51了，为什么？我们看下linux下运行的栈内存分布：

在strcpy之前的内存分布如下：

![stack1](/images/post/stack1.png)


linux的栈底为高地址，栈的增长方向从高地址到低地址。 因为auth_flag=0定义并初始化在password_buffer之前，
所以auth_flag先入栈，占用4个字节,并初始化为0， password_buffer 后入栈，所以在左边，占用12个字节， 为初始化，所以也不知道里面的具体数据。

strcpy之后：  

![stack2](/images/post/stack2.png)


拷贝password的时候，由于没有对传进来的password进行长度检查，就直接拷贝1234567890123，当拷贝到最后一个3的时候，password_buffer的12个字节已经用完，
这时候只能用了auth_flag的第一个字节， 所以auth_flag第一个字节是3对应的ANSI编码：51， 所以auth_flag 变成了51！ 

等等！ 为什么是51？ auth_flag不是4个字节吗？我们把51转成16进制不就是 0x33,那auth_flag应该是0x33000000, 十进制就是855631086 ！
然而确实是51，其实auth_flag在这里被当成了0x00000033来处理，至于为什么，不清楚的，请参看下面的“大小端”介绍。
linux和其他大部分操作系统的内存操作都是小端模式， 所以就很好的解释了auth_flag为嘛是0x00000033了。


然后再看下，为什么就通过验证了：

    if (check_authentication(argv[1]))

因为auth_flag=51，肯定是能验证通过了，那假如把代码改成下面这样：

    if (check_authentication(argv[1]) == 1) 

检查的结果必须为1才能验证通过，这时候我们遇到了难题，ANSI中1对应的是控制字符SOH，怎么输入？ 反正到最后都是二进制数据，那肯定是有办法的，比如：

    $ ./auth_overflow.o `php -r "echo '123456789012'.chr(1);"`
    password_buffer: 123456789012 # 这里最后其实有一个SOH字符编码
    password_buffer_size: 12
    auth_flag: 1

    -=-=-=-=-=-=-=-=-=-=-
     Access Granted.
    -=-=-=-=-=-=-=-=-=-=-

上面的命令使用php中的chr函数拿到1对应的ANSI控制字符soh, auth_flag成功被修改成了1。 

### 如何修复
1. 在strcpy的时候进行长度检查，这种是最常用的
2. 对于上面这个例子有个最简单的办法， 仔细看一下上面的内存分布图， 可以发现auth_flag之所以可以被修改，
其实就是因为入栈比password_buffer早，在它的右边，只要对调两行代码，就可以防止auth_flag不被修改：
    
        char password_buffer[12]; //存储在栈中，用来缓冲传进来的password
        int auth_flag = 0;  //存储在栈中
        
这样虽然auth_flag不会被修改，但是写入password_buffer的时候，还是可能导致其后面的内存被修改，所以，最好的做法还是进行长度检查，最多拷贝缓冲区长度的字符。
3. 去掉-fno-stack-protector,让gcc为我们处理,但是gcc会增加一些代码。

### 再扯一下另一种情况
我们还是保持修复之前的代码， 代码顺序不变， 也没有长度检查：

    int auth_flag = 0;  //存储在栈中
    char password_buffer[12]; //存储在栈中，用来缓冲传进来的password

把password_buffer的长度改为16,这个时候需要把输入密码加到多长才能验证成功？ 看内存分布图

![stack3](/images/post/stack3.png)

难道17就行了？ 哈， 上面的内存分布图是错的，正确的应该是这样的：

![stack4](/images/post/stack4.png)    

为什么中间会隔着没有使用的12字节？ 也就是说必须把密码延长到28+1才能修改auth_flag中的第一个字节！ 测试一下：

    $ ./auth_overflow.o 12345678901234567890123456781
    password_buffer: 12345678901234567890123456781
    password_buffer_size: 16
    auth_flag: 49

    -=-=-=-=-=-=-=-=-=-=-
     Access Granted.
    -=-=-=-=-=-=-=-=-=-=-

为什么？操作系统为了提高内存存储与读取速度，栈帧是要求16字节对齐的，每个栈帧的长度必须是16字节的倍数。不够16个字节也会补满，所以才会有中间空出的12字节。
    
### 什么是大小端
* Little-Endian就是低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。
* Big-Endian就是高位字节排放在内存的低地址端，低位字节排放在内存的高地址端。

举一个例子，比如数字0x12 34 56 78在内存中的表示形式为：

* 大端模式：

        低地址 -----------------> 高地址
        0x12  |  0x34  |  0x56  |  0x78
    
* 小端模式：

        低地址 ------------------> 高地址
        0x78  |  0x56  |  0x34  |  0x12

大端模式更符合人类的阅读习惯，小端刚好相反。可以用下面这段程序查看系统是采用哪种模式，大部分操作系统基本都是小端：

~~~c

    /**
     * endian check
     *
     * */

    int main (int argc, char *argv)
    {
        short int  x;
        char x0, x1;
        x = 0x1122;
        x0 = ((char*)&x)[0];
        x1 = ((char*)&x)[1];

        printf("x0 : %x \n", x0);
        printf("x1 : %x \n", x1);

        // x0=11 : big endian
        // x0=22 : little endian
        if (x0 == 0x11)
            printf("Big Endian!\n");
        else 
            printf("Little Endian!\n");
        return 0;
    }

~~~  

### 总结
这个例子涉及了几个知识点：
* 函数内部栈
* 栈生长方向， 高->低
* 内存对齐
* 大小端

要彻底了解栈溢出，这些是必须了解的，否则有些细节的地方就不知道为什么了。 看到这篇文章的同鞋，也可以试试。


[GCC堆栈保护原理]: https://www.ibm.com/developerworks/cn/linux/l-cn-gccstack/
