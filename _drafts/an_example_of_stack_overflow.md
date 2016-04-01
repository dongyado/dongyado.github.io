---
layout: post
title: linux下栈溢出的一个例子
date: 2016-03-31
categories:
- Linux
- Funny
tags: [Funny]
status: publish
type: post
published: true
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---

偶然看到一本书，中文名应该叫做黑客艺术，挺有意思的一本书，不是教人找漏洞，而是教人写漏洞。

反过来想一下，比起那种千篇一律，教人使用各种工具找漏洞的书，我觉得这种书才是从根本出发，也越容易让人接受。

### 什么是栈溢出
栈溢出，其实就是使用了不该使用的内存，在缓冲区的时候，如果忘记检查，就会出现溢出的问题，从而成为一个漏洞。


### 模拟栈溢出漏洞

我们使用C语言举例，我们知道函数里面的变量会存储在栈里面，我们写一个检查权限的函数：


    int check_authentication(char *password)
    {
        int auth_flag = 0;  //存储在栈中
        char password_buffer[16]; //存储在栈中，用来缓冲传进来的password

        printf("start copy\n");

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


然后还有一个入口函数：

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

把这两段代码加入到一个auth_overflow.c的文件里面，再加上一些必须的头信息：

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>

然后我们可以编译运行了:
    
    gcc -g -o auth_overflow.o auth_overflow.c 

运行

    ./auth_overflow.o password_text

但是其实里面有一个栈溢出的问题（先不管这个问题，运行起来再说）。
gcc编译的时候已经会把栈溢出的代码优化，所以编译的时候加上-fno-stack-protector, 禁用gcc的栈溢出保护优化：

    gcc -g -fno-stack-protector -o auth_overflow.o auth_overflow.c 

再测试：

    ./auth_overflow.o password 

正常的输入outgrade或者brillig测试,会输出 Access Granted
