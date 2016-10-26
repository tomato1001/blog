---
layout: post
title: tomcat-configuration
categories: tomcat
date: 2016-01-20 18:16:09 +0800
---

# tomcat7
{% highlight xml %}
<Connector port="80" protocol="org.apache.coyote.http11.Http11NioProtocol"
           connectionTimeout="60000"
           maxThreads="300"
           acceptCount="50"
           socket.appWriteBufSize="1024000"
           socket.txBufSize="1024000"
           socket.bufferPool="300"
           redirectPort="8443" URIEncoding="UTF-8" />
{% endhighlight %}
txBufSize与appWriteBufSize过大时将会出现大量请求错误

# tomcat8
使用nio2

{% highlight xml %}
<Connector port="80" protocol="org.apache.coyote.http11.Http11Nio2Protocol"
       connectionTimeout="60000"
       maxThreads="300"
       acceptCount="50"
       socket.bufferPool="300"
       redirectPort="8443" URIEncoding="UTF-8" />
{% endhighlight %}
