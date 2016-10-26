---
layout: page
title: ovsp-fasteditor
categories: others ovsp-fasteditor
date: 2016-01-25 14:33:58 +0800
---

- [简介](#introduction)
- [分析结果](#result)
  - [FEServiceImpl](#fe)

# 简介 {#introduction}
该文档内容是对ovsp-fasteditor项目的代码分析结果.代码路径: /ovsp-fasteditor/trunk/ovsp-fasteditor

# 分析结果 {#result}

## com.arcsoft.ovsp.service.impl.FEServiceImpl {#fe}
- 比较字符串时应使用**equals**方法
![fe-listOnlineContent](/images/ovsp-fasteditor/fes-listOnlineContent.png)

- 使用Integer.parseInt比valueOf更高效
![fe-loc](/images/ovsp-fasteditor/fe-loc.png)

- 字符串拼接
当需要进行字符串拼接且不需要线程安全时,优先使用StringBuilder

![fe-loc-2](/images/ovsp-fasteditor/fe-loc-2.png)

- 冗余的条件判断
![fe-loc-3](/images/ovsp-fasteditor/fe-loc-3.png)