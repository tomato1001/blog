---
layout: post
title:  "java进程cpu占用高排查"
date:   2015-05-12 22:51:00
categories: java tuning
---
最近在一个项目中遇到java进程cpu占用超高的问题，远程过去瞅了下，cpu占用2600%，初步断定肯定是哪里出现死循环了。既然已经知道了大致的问题，下面就得定位具体代码了，排查过程如下：  

* 首先查看java进程中各线程cpu占用情况  
  `ps -mp 28899 -o THREAD,tid,time`  

  > **28899**为java进程  
* 找到cpu占用高的线程id后，转换为16进制值  
  `printf "%x\n"` 15278  

  > 15278为线程id  
* 使用jdk提供的命令jstack查看线程栈调用  
  `jstack 28899 | grep 0x3bae -A 100`

  > 0x3bae为上一步获取到的进程id的16进制值  
  > -A可以指定输出多少行  

获得线程堆栈后就可以根据调用过程初步判断问题代码了，这次遇到的问题是aspectj 1.5.4版本中使用WeakHashMap时没有同步，在并发调用resize时进入死循环了.
