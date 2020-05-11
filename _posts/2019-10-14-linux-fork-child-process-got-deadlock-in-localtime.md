---
layout: post
title: Linux localtime/localtime_r 导致 fork 子进程死锁卡死问题的研究
date: 2019-10-14
categories:
- linux
tags: [process, localtime, fork]
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  views: '2'
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---

### 问题背景

ve 短视频渲染服务程序基于 opengl 和 c/c++ 开发， 服务器渲染现阶段只支持 linux，渲染流程是： 父进程下载数据和素材，再 fork 一个渲染子进程进行渲染。

近期发现生产环境中，极小概率下，会出现渲染子进程假死、卡住以及无任何日志输出的现象。

这个问题在现有生产流程下很难重现，最初采用的解决方法是在父进程中检测渲染进程启动失败时，发送信号 Kill 掉后重试任务。

最近特意抽出时间仔细分析并解决了这个问题，觉得很有意思，记录下来。

### 问题查找

#### 重现问题

渲染服务由于需要预先下载素材和准备数据，很难达到同时启动多个任务的情况，所以特意重写了多线程支持本地多任务的测试程序。

基本流程就是同一时间启动多个线程，每个线程 fork 一个子进程进行渲染，然后等待所有线程完成。

经过多次测试重现假死的概率很高。

#### 定位问题

稳定重现问题是定位问题的基本条件，在稍微修改测试程序后，使用 debug 编译，正常启动后就可以使用 gdb 调试，找到进程具体卡死的位置，步骤如下：

```shell
slayer@slayer-B250M-D3H:~$ sudo gdb -p 12680
GNU gdb (Ubuntu 7.11.1-0ubuntu1~16.5) 7.11.1
Copyright (C) 2016 Free Software Foundation, Inc.
...
[Thread debugging using libthread_db enabled]
...
(gdb) info threads
  Id   Target Id         Frame 
* 1    Thread 0x7ff6287a6700 (LWP 12680) "SXVideoEngineDe" __lll_lock_wait_private ()
    at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:95
(gdb) thread 1
[Switching to thread 1 (Thread 0x7ff6287a6700 (LWP 12680))]
#0  __lll_lock_wait_private () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:95
95	in ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S
(gdb) where
#0  __lll_lock_wait_private () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:95
#1  0x00007ff6421bfc76 in __tz_convert (timer=0x7ff6424c8ac0 <tzset_lock>, use_localtime=1, tp=0x7ff6287a39d0)
    at tzset.c:616
#2  0x00000000013b2c01 in el::base::utils::DateTime::buildTimeInfo (currTime=0x7ff6287a39a0, timeInfo=0x7ff6287a39d0)
    at /home/slayer/workspace/SXVideoEngine-Core/Render/log/easylogging++.cc:1232
#3  0x00000000013b2751 in el::base::utils::DateTime::timevalToString[abi:cxx11](timeval, char const*, el::base::SubsecondPrecision const*) (tval=..., format=0x37bdbb0 "%Y-%M-%d %H:%m:%s,%g", ssPrec=0x37f2cdc)
    at /home/slayer/workspace/SXVideoEngine-Core/Render/log/easylogging++.cc:1190
#4  0x00000000013b26da in el::base::utils::DateTime::getDateTime[abi:cxx11](char const*, el::base::SubsecondPrecision const*) (format=0x37bdbb0 "%Y-%M-%d %H:%m:%s,%g", ssPrec=0x37f2cdc)
    at /home/slayer/workspace/SXVideoEngine-Core/Render/log/easylogging++.cc:1184
#5  0x00000000013ba36c in el::base::DefaultLogBuilder::build[abi:cxx11](el::LogMessage const*, bool) const (
    this=0x37b9ff0, logMessage=0x7ff6287a4030, appendNewLine=true)
    at /home/slayer/workspace/SXVideoEngine-Core/Render/log/easylogging++.cc:2412
#6  0x00000000013b9d94 in el::base::DefaultLogDispatchCallback::handle (this=0x37f8b20, data=0x7ff6287a3ee0)
    at /home/slayer/workspace/SXVideoEngine-Core/Render/log/easylogging++.cc:2212
#7  0x00000000013bb2cf in el::base::LogDispatcher::dispatch (this=0x7ff6287a3f70)
    at /home/slayer/workspace/SXVideoEngine-Core/Render/log/easylogging++.cc:2499
#8  0x00000000013bc0b5 in el::base::Writer::triggerDispatch (this=0x7ff6287a42e0)
    at /home/slayer/workspace/SXVideoEngine-Core/Render/log/easylogging++.cc:2632
#9  0x00000000013bbdf8 in el::base::Writer::processDispatch (this=0x7ff6287a42e0)
    at /home/slayer/workspace/SXVideoEngine-Core/Render/log/easylogging++.cc:2613
#10 0x00000000004392b8 in el::base::Writer::~Writer (this=0x7ff6287a42e0, __in_chrg=<optimized out>)
    at /home/slayer/workspace/SXVideoEngine-Core/Render/log/easylogging++.h:3213
#11 0x00000000013aa518 in VEPrintLog (level=3, str=0x1ab9448 "Child process ve_render_process_start() : %s", 
    args=0x7ff6287a53a0) at /home/slayer/workspace/SXVideoEngine-Core/Render/log/corelog.cpp:135
#12 0x00000000013aa0ad in VELogInfo (str=0x1ab9448 "Child process ve_render_process_start() : %s")
    at /home/slayer/workspace/SXVideoEngine-Core/Render/log/corelog.cpp:90
#13 0x000000000180b09d in ve_render_process_start (id=0x7ff6287a5740, 
    handler=0x43abb5 <MultiThreadVideoTest::Handler(unsigned long*, char const*)>)
    at /home/slayer/workspace/SXVideoEngine-Core/Render/server/RenderProcess.cpp:597
#14 0x000000000043acea in MultiThreadVideoTest::RenderThread (context=0x7ffe95b37320)
    at /home/slayer/workspace/SXVideoEngine-Core/Render/SXVideoEngineDemo.cpp:342
#15 0x00007ff6470e46ba in start_thread (arg=0x7ff6287a6700) at pthread_create.c:333
#16 0x00007ff64220941d in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:109
(gdb) 

```

确定死锁卡死的点为：

```
#0  __lll_lock_wait_private () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:95
#1  0x00007ff6421bfc76 in __tz_convert (timer=0x7ff6424c8ac0 <tzset_lock>, use_localtime=1, tp=0x7ff6287a39d0)
    at tzset.c:616
#2  0x00000000013b2c01 in el::base::utils::DateTime::buildTimeInfo (currTime=0x7ff6287a39a0, timeInfo=0x7ff6287a39d0)
    at /home/slayer/workspace/SXVideoEngine-Core/Render/log/easylogging++.cc:1232
```

查看源码，确定 easylogging 在打印日志需要格式化当前时间，调用了 localtime_r 的方法， 查看 localtime.c 源码，

函数调用栈为： localtime_r -> __localtime_r -> __localtime64_r -> __tz_convert 

```c
struct tm *
__localtime64_r (const __time64_t *t, struct tm *tp)
{
  return __tz_convert (*t, 1, tp);
}
/* Provide a 32-bit variant if needed.  */
#if __TIMESIZE != 64
struct tm *
__localtime_r (const time_t *t, struct tm *tp)
{
  __time64_t t64 = *t;
  return __localtime64_r (&t64, tp);
}
libc_hidden_def (__localtime64_r)
#endif
weak_alias (__localtime_r, localtime_r)

```

继续查看 __tz_convert 的源码， 在 tzset.c 中, 可以看到调用了    `__libc_lock_lock (tzset_lock);` 加锁，进程就是死锁在这里。

```c
/* Return the `struct tm' representation of TIMER in the local timezone.
   Use local time if USE_LOCALTIME is nonzero, UTC otherwise.  */
struct tm *
__tz_convert (__time64_t timer, int use_localtime, struct tm *tp)
{
  long int leap_correction;
  int leap_extra_secs;
  __libc_lock_lock (tzset_lock);
  /* Update internal database according to current TZ setting.
     POSIX.1 8.3.7.2 says that localtime_r is not required to set tzname.
     This is a good idea since this allows at least a bit more parallelism.  */
  tzset_internal (tp == &_tmbuf && use_localtime);
  if (__use_tzfile)
    __tzfile_compute (timer, use_localtime, &leap_correction,
                      &leap_extra_secs, tp);
  else
    {
      if (! __offtime (timer, 0, tp))
        tp = NULL;
      else
        __tz_compute (timer, tp, use_localtime);
      leap_correction = 0L;
      leap_extra_secs = 0;
    }
  __libc_lock_unlock (tzset_lock);
  if (tp)
    {
      if (! use_localtime)
        {
          tp->tm_isdst = 0;
          tp->tm_zone = "GMT";
          tp->tm_gmtoff = 0L;
        }
      if (__offtime (timer, tp->tm_gmtoff - leap_correction, tp))
        tp->tm_sec += leap_extra_secs;
      else
        tp = NULL;
    }
  return tp;
}
```

  `__libc_lock_lock (tzset_lock);` 执行了加锁操作， tzset_lock 是通过下面的宏定义的


```c
__libc_lock_define_initialized (static, tzset_lock)
```
宏替换后的代码是 

```c
static __libc_lock_t tzset_lock;
```

最终在 libc-lock.h 中找到 __libc_lock_t 的定义

```c
typedef pthread_mutex_t __libc_lock_t;
```

可以确认这是一把静态共享锁。

+ 为什么这里需要加锁？

__tz_convert 中计算的时候使用了多个全局变量，比如 _tm_buf, __tz_name, __daylight, __timezone 等全局变量，计算过程中，都有读写，非线程安全。

+ 为什么会产生死锁？

fork 时，子进程会复制锁和锁的状态，假如复制时，主进程的刚好有个操作进行了加锁操作，此时子进程得到的锁就已经是**上锁**的状态，这时子进程再去加锁，只会永远等待，因为没有任何操作去解锁；主进程中解锁也是解自己的锁，与子进程无关。

这样就得到的死锁的原因和根源，剩下的就是找到最合适的解决方法。


### 解决方法

办法有很多，最终选择替换 localtime_r 函数，参考了 redis 5.0 实现的 nolocks_localtime 函数，彻底替换了 localtime_r 函数。

参考 https://github.com/antirez/redis/blob/5.0.0/src/localtime.c

### 总结

多线程且有锁操作的程序要谨慎使用 fork, 存在父进程和子进程都要使用的锁，复制后都有可能造成子进程死锁。

最终总结一下解决思路：

1. 避免共享锁的代码
2. 在 fork 之后立即调用 execv 方法抛弃父进程的空间数据，启用新的进程内存空间，从根本上独立于父进程，相当重新开了个进程，自然不会有锁的问题
3. 使用 pthread_afork 在fork 前释放共享的锁，这个方法使用并不方便，如果引用的库中包含锁，很难对全部的锁进行操作，解锁时也可能导致使用锁的程序状态异常
