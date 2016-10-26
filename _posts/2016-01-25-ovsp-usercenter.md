---
layout: page
title: ovsp-user
categories: others ovsp-user
date: 2016-01-25 17:00:10 +0800
---

- [简介](#introduction)
- [高](#high)
  - [字符串比较](#str-equals)
  - [内存泄漏](#memory-leak)
- [中](#middle)
  - [命名不规范](#illegal-name)
  - [兼容性](#compatibility)
    - [默认编码](#default-encoding)  
  - [性能问题](#performance)
    - [字符串拼接](#string-concate)
    - [StringBuffer](#stringbuffer)
    - [异常处理](#exception)
- [低](#low)
  - [设计相关](#structure)

# 简介 {#introduction}
该文档内容是对ovsp-user项目的代码分析结果.代码路径: /ovsp-user/trunk/usercenter

分析结果分为以下等级:

* 高
代码误用或编码不当导致的问题以及潜在的隐患.建议立即修复

* 中
主要分析api使用方面的问题,建议修复

* 低
主要分析程序结构与设计方面的问题

# 高 {#high}

## 字符串比较 {#str-equals}
比较字符串时应使用equals
![tc-1](/images/ovsp-usercenter/tc-1.png)

## 内存泄漏 {#memory-leak}
以下代码在进行I/O操作时,没有调用**close**方法.

- com.arcsoft.ovsp.usercenter.oss.AliYun
  - uploadFile
  ![ay-1](/images/ovsp-usercenter/ay-1.png)

下面的代码没有在异常情况下释放资源,当出现异常后会导致内存泄漏

- com.arcsoft.ovsp.usercenter.util.ComputeMessageDigest
  - getMd5OfFile(String filePath)
  ![cmd-1](/images/ovsp-usercenter/cmd-1.png)

  - getMd5OfFile(File file)
  ![cmd-2](/images/ovsp-usercenter/cmd-2.png)  

建议使用jdk7的try-with-resources语法简化IO操作并确保资源正确释放.如:

{% highlight java %}
try(InputStream input = new FileInputStream(file)) {
...
}
{% endhighlight %}

# 中 {#middle}

## 命名不规范 {#illegal-name}
根据java命名规范,方法名首字母应小写且以动词开头

- com.arcsoft.ovsp.usercenter.util.OauthUtil
![ou-1](/images/ovsp-usercenter/ou-1.png)

- com.arcsoft.ovsp.usercenter.util.Util
  - IgnoreUserProperty
  ![u-1](/images/ovsp-usercenter/u-1.png)

  - IgnoreVideoProperty
  ![u-2](/images/ovsp-usercenter/u-2.png)

## 兼容性 {#compatibility}

### 默认编码 {#default-encoding}
下面代码中使用默认编码,当默认编码不一致时,将会产生不同的结果

- com.arcsoft.ovsp.usercenter.transcode.CloudTranscode
  - createTranscode
  - createUnencryptTranscode
  - createTrailorTranscode
  - createTaskTemplateForFastEdit
  - createTranscodeForFastEdit
  ![ct-1](/images/ovsp-usercenter/ct-1.png)

## 性能问题 {#performance}

### 字符串拼接 {#string-concate}
当需要进行字符串拼接且不需要线程安全时,优先使用StringBuilder

- com.arcsoft.ovsp.usercenter.util.ComputeMessageDigest
  - getMd5OfFile(String filePath)
  ![cmd-3](/images/ovsp-usercenter/cmd-3.png)

  - getMd5OfFile(File file)
  ![cmd-4](/images/ovsp-usercenter/cmd-4.png)

- com.arcsoft.ovsp.usercenter.util.Util
  - replaceString
  ![u-4](/images/ovsp-usercenter/u-4.png)  

### StringBuffer {#stringbuffer}
因**StringBuffer**是一个线程安全的类,内部使用了同步机制,所以在使用时会带来一些开销.当不需要线程安全时,可以直接使用StringBuilder,从而提升性能

### 异常处理 {#exception}
很多class中在处理异常时,直接使用**e.printStackTrace()**输出异常栈,可以考虑将异常栈输出到log中,如:

{% highlight java %}
log.error("xxxx", e);
{% endhighlight %}

# 低 {#low}

## 设计相关 {#structure}

- com.arcsoft.ovsp.usercenter.transcode.CloudTranscode
该class用于生成task的xml,因xml节点比较多,且代码与xml节点信息混在一起,导致每个方法都在100多行以上,有些方法行数已达到300多行.
建议将xml信息抽象为view,数据为model,从而实现MVC模式,达到数据与xml视图分离的目的,减少开发及维护成本.比如:使用模板库,将xml节点信息作为模板,在生成xml时,将数据传递给模板并获得结果.

