---
layout: post
title:  "Tomcat https不能访问"
date:   2015-05-13 23:08:00
categories: tomcat
---
首先确认配置是否正确，如果配置正确且无法访问https，尝试删除bin目录下的***tcnative-1.dll文件***.或者在server.xml中以下代码注释：

{% highlight xml %}
<Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
{% endhighlight %}

另外一种解决方案是，参考apache tomcat官网的apr配置，对http、ssl等进行配置。
