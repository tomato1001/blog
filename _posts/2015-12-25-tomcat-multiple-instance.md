---
layout: post
title: tomcat-multiple-instance
categories: ['tomcat']
date: 2015-12-25 17:27:22
---

多实例即使用同一个tomcat,启动多个不同的进程.

* 将CATALINA_BASE的目录设置到不同的目录后,再启动tomcat.
* 创建app
	- app1
		新建app1目录,并将tomcat目录下的conf,temp,webapps,logs文件夹复制到app1目录
	- app2
		新建app2目录,并将tomcat目录下的conf,temp,webapps,logs文件夹复制到app2目录

* 分别修改app1和app2目录下的conf/server.xml,将端口修改为不同的端口.
* 启动和停止时,将CATALINA_BASE设置到当前app目录,调用startup.sh或shutdown.sh开始或关闭当前的tomcat.

{% highlight sh %}
startup.sh
#!/bin/sh

export CATALINA_BASE="$(pwd)"
/usr/local/tomcat/bin/startup.sh

shutdown.sh

#!/bin/sh

export CATALINA_BASE="$(pwd)"
/usr/local/tomcat/bin/shutdown.sh
{% endhighlight %}