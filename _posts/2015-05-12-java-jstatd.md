---
layout: post
title: "jstatd使用visualvm进行远程监控"
date: 2015-05-13 23:00:00
categories: java tuning
---
* 先开启rmi：

{% highlight sh %}
rmiregistry 2020
{% endhighlight %}

* 开启jstatd：

{% highlight sh %}
jstatd -J-Djava.security.policy=policy -p 2020
{% endhighlight %}
> * policy是安全文件，内容如下：  
> 	grant codebase "file:/usr/local/jswzjhc/jdk1.5.0_22/lib/tools.jar" {  
   	    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;permission java.security.AllPermission;  
	};

* 指定ip地址

{% highlight sh %}
jstatd -J-Djava.rmi.server.hostname=192.168.8.7 -J-Djava.security.policy=all.policy
{% endhighlight %}


