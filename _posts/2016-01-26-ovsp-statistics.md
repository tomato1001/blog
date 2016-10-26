---
layout: page
title: ovsp-statistics
categories: others ovsp-statistics
date: 2016-01-26 09:09:16 +0800
---

- [简介](#introduction)
- [分析结果](#result)
  - [SysTrafficDaoImpl](#stdi)
  - [SysSpaceTrafficMintor](#sstm)  

# 简介 {#introduction}
该文档内容是对ovsp-statistics项目的代码分析结果.代码路径: /ovsp-statistics/trunk/ovsp-statistics

# 分析结果 {#result}

## com.arcsoft.ovsp.dao.impl.SysTrafficDaoImpl {#stdi}
- 冗余的**String.format**或格式化字符串忘记了?
![stdi](/images/ovsp-statistics/stdi-1.png)

## com.arcsoft.ovsp.traffic.SysSpaceTrafficMintor {#sstm}

- 忽略的返回值
以下代码在创建目录时完全忽略了返回值,至少保证针对错误的返回值将其输出到log,方便后期排查?

![sstm-1](/images/ovsp-statistics/sstm-1.png)

- 线程安全
![sstm-2](/images/ovsp-statistics/sstm-2.png)

**SimpleDateFormat**是一个线程不安全的class,不能在多线程环境中使用

![sstm-3](/images/ovsp-statistics/sstm-3.png)

**maxStatisticsTime**是一个静态的全局变量,在分配值时没有进行同步处理.如果多个线程同时访问,将导致程序逻辑不正常.