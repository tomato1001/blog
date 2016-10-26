---
layout: page
title: ovsp-uploadserver
categories: others ovsp-uploadserver
date: 2016-01-25 16:05:01 +0800
---

- [简介](#introduction)
- [高](#high)
  - [内存泄漏](#memory-leak)
- [中](#middle)
  - [忽略的返回值](#m-ignore-rvalue)
  - [兼容性](#compatibility)
    - [默认编码](#default-encoding)  
  - [命名不规范](#illegal-name)
  - [性能问题](#performance)
    - [Map](#map)
    - [字符串拼接](#string-concate)
    - [Integer.valueOf](#ivo)
    - [StringBuffer](#stringbuffer)
    - [异常处理](#exception)
    - [System.out](#sysout)

# 简介 {#introduction}
该文档内容是对ovsp-uploadserver项目的代码分析结果.代码路径: /ovsp-upload/trunk/ovsp-uploadserver

分析结果分为以下等级:

* 高
代码误用或编码不当导致的问题以及潜在的隐患.建议立即修复

* 中
主要分析api使用方面的问题,建议修复

* 低
主要分析程序结构与设计方面的问题

# 高 {#high}

## 内存泄漏 {#memory-leak}
以下代码在进行I/O操作时,没有调用**close**方法.

- com.arcsoft.ovsp.uploadserver.oss.AliYun
  - uploadFile
  ![ay-2](/images/ovsp-uploadserver/ay-2.png)

- com.arcsoft.ovsp.uploadserver.util.FileUtil
  - uploadToOSSByFile
  ![fu-4](/images/ovsp-uploadserver/fu-4.png)

建议使用jdk7的try-with-resources语法简化IO操作并确保资源正确释放.如:

{% highlight java %}
try(InputStream input = new FileInputStream(file)) {
...
}
{% endhighlight %}

下面的代码没有在异常情况下释放资源,当出现异常后会导致内存泄漏

- com.arcsoft.ovsp.uploadserver.util.FileUtil
  - saveFile
  ![fu-5](/images/ovsp-uploadserver/fu-5.png)
  - MergeRunnable.run
  ![fu-6](/images/ovsp-uploadserver/fu-6.png)

- com.arcsoft.ovsp.uploadserver.util.HashUtil
  - getMd5OfFile
  ![hu-1](/images/ovsp-uploadserver/hu-1.png)

- com.arcsoft.ovsp.uploadserver.util.VideoUtils
  - getVideoThumb
  ![vu-1](/images/ovsp-uploadserver/vu-1.png)

# 中 {#middle}

## 忽略的返回值 {#m-ignore-rvalue}
以下代码在创建或删除文件时完全忽略了返回值,至少保证针对错误的返回值将其输出到log,方便后期排查?

- com.arcsoft.ovsp.uploadserver.task.DownloadFromOssTask#call
![dfot-1](/images/ovsp-uploadserver/dfot-1.png)

- com.arcsoft.ovsp.uploadserver.task.UploadThread#run
![ut-1](/images/ovsp-uploadserver/ut-1.png)

- com.arcsoft.ovsp.uploadserver.util.FileUtil
  - mergeFile
  ![fu-1](/images/ovsp-uploadserver/fu-1.png)<br>
  ![fu-2](/images/ovsp-uploadserver/fu-2.png)

  - createFileIfNotExist
  ![fu-3](/images/ovsp-uploadserver/fu-3.png)

## 兼容性 {#compatibility}

### 默认编码 {#default-encoding}
<ul>
  <li id="h-c-hmacSha">com.arcsoft.ovsp.uploadserver.util.SignatureUtil
    <ul>
        <li id="h-c-hmacSha-HmacSHA256Encrypt">HmacSHA256Encrypt<br>下面代码中的<code>new String(hexB)</code>使用默认编码,当默认编码不一致时,将会产生不同的结果<br>
  <img src="/images/ovsp-api/hamcsha-sha256-encrypt.png" alt="HmacSHA256Encrypt"></li>
    </ul>
  </li>
</ul>

## 命名不规范 {#illegal-name}
根据java命名规范,方法名首字母应小写且以动词开头

- com.arcsoft.ovsp.uploadserver.util.CodeUtil
![cu-1](/images/ovsp-uploadserver/cu-1.png)

- com.arcsoft.ovsp.uploadserver.util.SignatureUtil
![su-1](/images/ovsp-uploadserver/su-1.png)

## 性能问题 {#performance}

### Map {#map}
当需要同时获取Map的key和value时,应使用entrySet(),如:

{% highlight java %}
for(Entry<String, List<MultipartFile>> entry: fileMap.entrySet()) {
	String key = entry.getKey();
	List<MultipartFile> value = entry.getValue();
}
{% endhighlight %}
下面的代码通过循环keySet得到key,然后在循环中get对应的key获取value.

- com.arcsoft.ovsp.uploadserver.http.HttpClientUtil
  - assmbleParams(String url, Map<String,Object> params)
  ![hcu-1](/images/ovsp-uploadserver/hcu-1.png)

  - assmbleParams(Map<String,Object> params)
  ![hcu-2](/images/ovsp-uploadserver/hcu-2.png)

- com.arcsoft.ovsp.uploadserver.http.SignatureUtil
  - generateParams
  ![su-2](/images/ovsp-uploadserver/su-2.png)

### 字符串拼接 {#string-concate}
当需要进行字符串拼接且不需要线程安全时,优先使用StringBuilder

- com.arcsoft.ovsp.uploadserver.http.HttpClientUtil
  - assmbleParams(String url, Map<String,Object> params)
  ![hcu-3](/images/ovsp-uploadserver/hcu-3.png)

- com.arcsoft.ovsp.uploadserver.http.HashUtil
  - getMd5OfFile
  ![hu-2](/images/ovsp-uploadserver/hu-2.png)

### Integer.valueOf {#ivo}
将字符串转换为Integer时,使用Integer.parseInt更高效
![ru-1](/images/ovsp-uploadserver/ru-1.png)

### StringBuffer {#stringbuffer}
因**StringBuffer**是一个线程安全的类,内部使用了同步机制,所以在使用时会带来一些开销.当不需要线程安全时,可以直接使用StringBuilder,从而提升性能

### 异常处理 {#exception}
很多class中在处理异常时,直接使用**e.printStackTrace()**输出异常栈,可以考虑将异常栈输出到log中,如:

{% highlight java %}
log.error("xxxx", e);
{% endhighlight %}

### System.out {#sysout}
某些class中包含一些**System.out.println**,建议输出到log,或者直接删除.

