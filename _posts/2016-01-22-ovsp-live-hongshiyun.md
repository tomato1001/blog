---
layout: page
title: ovsp-live-hongshiyun
categories: others ovsp-live-hongshiyun
date: 2016-01-22 10:35:57 +0800
---

- [简介](#introduction)
- [高](#high)
  - [内存泄漏](#memory-leak)
    - [FileUtil](#h-ml-fl)
      - [uploadToOSSByFileAndBucketName](#h-ml-fl-utobfabn)
      - [uploadToOSSByFile](#h-ml-fl-utf)
    - [LiveThumbnailUtil](#h-ml-ltu)
      - [genearteThumbnail](#h-ml-ltu-genearteThumbnail)
    - [HttpUtil](#h-ml-hu)
      - [postJson](#h-ml-hu-postJson)
- [中](#middle)
  - [忽略的返回值](#m-ignore-rvalue)
    - [FileUtil](#m-ignore-rvalue-fu)
      - [createFileIfNotExist](#m-ignore-rvalue-fu-cfine)
  - [命名不规范](#illegal-name)
    - [Util](#m-ln-util)
  - [数据类型](#data-type)
  - [性能问题](#performance)
    - [Map](#map)
      - [HttpClientUtil](#m-p-hu)
        - [assmbleParams](#m-p-hu-ap)
        - [assmbleParams](#m-p-hu-ap2)        
    - [字符串拼接](#string-concate)
      - [HttpClientUtil](#m-sc-hcu)
        - [assmbleParams](#m-sc-hcu-ap)
      - [MD5](#m-sc-md5)
        - [byte2hex](#m-sc-md5-b2)
      - [Util](#m-sc-util)
        - [replaceString](#m-sc-util-rs)         
    - [StringBuffer](#stringbuffer)
    - [System.out](#sysout)
- [低](#low)
  - [方法参数过多](#to-many-arg)
  - [重复代码](#l-repeat-code)
  - [设计相关](#structure)

# 简介 {#introduction}
该文档内容是对ovsp-live-hongshiyun项目的代码分析结果.代码路径: /ovsp-live/trunk/ovsp-live-hongshiyun

分析结果分为以下等级:

* 高
代码误用或编码不当导致的问题以及潜在的隐患.建议立即修复

* 中
主要分析api使用方面的问题,建议修复

* 低
主要分析程序结构与设计方面的问题

# 高 {#high}

## 内存泄漏 {#memory-leak}
  - <p id="h-ml-fl">com.arcsoft.ovsp.live.util.FileUtil<br>下面的代码没有关闭InputStream,会导致内存泄漏
  	<ul>
	  	<li id="h-ml-fl-utobfabn">uploadToOSSByFileAndBucketName<br>
		<img src="/images/ovsp-live-hsy/fileutil-uploadToOSSByFileAndBucketName.png"></li>
	  	<li id="h-ml-fl-utf">uploadToOSSByFile<br>
		<img src="/images/ovsp-live-hsy/fileutil-uploadToOssByFile.png"></li>		
	</ul>
    </p>

建议使用jdk7的try-with-resources语法简化IO操作并确保资源正确释放.如:

{% highlight java %}
try(InputStream input = new FileInputStream(file)) {
...
}
{% endhighlight %}

下面的代码没有在异常情况下释放资源,当出现异常后会导致内存泄漏

<ul>
  <li id="h-ml-ltu">com.arcsoft.ovsp.live.util.LiveThumbnailUtil
    <ul>
      <li>
        	<p id="h-ml-ltu-genearteThumbnail">genearteThumbnail<br>
	<img src="/images/ovsp-live-hsy/livethumbnailutil-generateThumbnail.png"></p>
      </li>
    </ul>
  </li>
  <li id="h-ml-hu">com.arcsoft.ovsp.live.http.HttpUtil
    <ul>
      <li>
        	<p id="h-ml-hu-postJson">postJson<br>
	<img src="/images/ovsp-live-hsy/httputil-postJson.png"></p>
      </li>
    </ul>
  </li>
</ul>

# 中 {#middle}

## 忽略的返回值 {#m-ignore-rvalue}
<ul>
  <li>
    <p id="m-ignore-rvalue-fu">com.arcsoft.ovsp.live.util.FileUtil<br>以下代码在创建目录时完全忽略了返回值,至少保证失败后将错误信息输出到日志,方便出错后排查?</p>
    <ul>
      <li>
        	<p id="m-ignore-rvalue-fu-cfine">createFileIfNotExist<br>
	<img src="/images/ovsp-live-hsy/fileutil-createFileIfNotExist.png" alt="createFileIfNotExist"></p>
      </li>
    </ul>
  </li>
</ul>

## 命名不规范 {#illegal-name}
<ul>
  <li id="m-ln-util">com.arcsoft.ovsp.live.util.Util<br>根据java命名规范,方法名首字母应小写且以动词开头<br>
  <img src="/images/ovsp-live-hsy/util-ignoreUserProperty.png" alt="illegal-name"></li>
</ul>

## 数据类型 {#data-type}
下面的代码在**Constants**中定义一些类型为Integer的常量.然后使用**==**来判断是否相等.
![constants-live-output](/images/ovsp-live-hsy/constants-live-output.png)

![constants-live-output2](/images/ovsp-live-hsy/constants-live-output-2.png)

![constants-live-output3](/images/ovsp-live-hsy/constants-live-output-3.png)

上面的liveOutputGroup的protocolType类型为Integer,当使用不当时将导致条件块内的语句永远不会被执行.比如:将protocolType设置为**new Integer(1)**

建议将**Constants**中定义的常量类型改为原始数据类型,让jvm做自动转换.从而避免因使用不当而导致的问题.

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
<ul>
  <li id="m-p-hu">com.arcsoft.ovsp.live.http.HttpClientUtil
    <ul>
      <li id="m-p-hu-ap">assmbleParams<br>
      	<img src="/images/ovsp-live-hsy/hcu-ap.png">
      </li>
    </ul>
      <li id="m-p-hu-ap2">assmbleParams<br>
      	<img src="/images/ovsp-live-hsy/hcu-ap-2.png" >
      </li>
    </ul>    
  </li>
</ul>

### 字符串拼接 {#string-concate}
当需要进行字符串拼接且不需要线程安全时,优先使用StringBuilder

<ul>
  <li id="m-sc-hcu">com.arcsoft.ovsp.live.http.HttpClientUtil
    <ul>
      <li id="m-sc-hcu-ap">
        <p>assmbleParams<br>
      <img src="/images/ovsp-live-hsy/hcu-assmbleParams.png"></p>
      </li>
    </ul>
  </li>
  <li id="m-sc-md5">com.arcsoft.ovsp.live.util.MD5
    <ul>
      <li id="m-sc-md5-b2">
        <p>byte2hex<br>
      <img src="/images/ovsp-live-hsy/md5-b2.png"></p>
      </li>
    </ul>
  </li> 
  <li id="m-sc-util">com.arcsoft.ovsp.live.util.Util
    <ul>
      <li id="m-sc-util-rs">
        <p>replaceString<br>
      <img src="/images/ovsp-live-hsy/util-rs.png"></p>
      </li>
    </ul>
  </li>   
</ul>

### StringBuffer {#stringbuffer}
因**StringBuffer**是一个线程安全的类,内部使用了同步机制,所以在使用时会带来一些开销.当不需要线程安全时,可以直接使用StringBuilder,从而提升性能

### System.out {#sysout}
某些class中包含一些**System.out.println**,建议输出到log,或者直接删除.

* com.arcsoft.ovsp.live.transcode.LiveTranscode#createArchiveTask
* com.arcsoft.ovsp.live.transcode.LiveTranscode#createLiveHlsTask
* com.arcsoft.ovsp.live.transcode.LiveTranscode#createLiveRtmpTask
* com.arcsoft.ovsp.live.transcode.LiveTranscode#stopAndDeleteTask
* com.arcsoft.ovsp.live.http.HttpUtil#postJson

# 低 {#low}

## 方法参数过多 {#to-many-arg}
在一些service中,有些方法包含大量的参数,考虑将这些参数封装成pojo对象?
![lts](/images/ovsp-live-hsy/lts.png)

![lts](/images/ovsp-live-hsy/lls.png)

## 重复代码 {#l-repeat-code}
- com.arcsoft.ovsp.live.util.FileUtil
![fileutil-cfine](/images/ovsp-live-hsy/fileutil-cfine.png)
考虑使用递归方式来减少重复代码?

## 设计相关 {#structure}

- com.arcsoft.ovsp.live.transcode.LiveTranscode
该class用于生成task的xml,因xml节点比较多,且代码与xml节点信息混在一起,导致每个方法都在100多行以上,**createLiveHlsTask**方法大概有267行.
建议将xml信息抽象为view,数据为model,从而实现MVC模式,达到数据与xml视图分离的目的,减少开发及维护成本.比如:使用模板库,将xml节点信息作为模板,在生成xml时,将数据传递给模板并获得结果.
