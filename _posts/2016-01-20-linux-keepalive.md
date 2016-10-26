---
layout: page
title: linux-tcp-keepalive
categories: linux tcp-keepalive
date: 2016-01-20 16:47:45 +0800
---

In Linux the TCP keepalive parameters are:

{% highlight sh %}
tcp_keepalive_intvl
tcp_keepalive_probes
tcp_keepalive_time
{% endhighlight %}

The default values are:

{% highlight sh %}
tcp_keepalive_time = 7200 (seconds)
tcp_keepalive_intvl = 75 (seconds)
tcp_keepalive_probes = 9 (number of probes)
{% endhighlight %}

This means that the keepalive process waits for two hours (7200 secs) for socket activity before sending the first keepalive probe, and then resend it every 75 seconds. If no ACK response is received for nine consecutive times, the connection is marked as broken. 

For example, to reduce the value of /proc/sys/net/ipv4/tcp_keepalive_time from 7,200 (2 hours) to 900 (15 Min) we can use  the command: 
 
{% highlight sh %}
echo 900> /proc/sys/net/ipv4/tcp_keepalive_time 
{% endhighlight %}

On many flavors of Linux the tool to manipulate TCP Keepalive on Linux is sysctl.

Example:

{% highlight sh %}
sysctl -w net.ipv4.tcp_keepalive_time=30 
{% endhighlight %}
 
The above two methods of changing TCP Keepalive are temporary and will only last until the system is rebooted.

A more permanent change to TCP Keepalive will require a change to the /etc/sysctl.conf file.
Example:
To make a permanent change to decrease TCP Keepalive time before testing to 30 seconds, edit /etc/sysctl.conf and add: 
 
{% highlight sh %}
net.ipv4.tcp_keepalive_time=30 
{% endhighlight %}
