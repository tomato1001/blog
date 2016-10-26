---
layout: page
title: centos-vagrant
categories: vagrant centos
date: 2016-03-28 13:55:41 +0800
---

首先下载vagrant.rpm

{% highlight sh %} 
wget https://releases.hashicorp.com/vagrant/1.8.1/vagrant_1.8.1_x86_64.rpm -O vagrant_1.8.1_x86_64.rpm --no-check-certificate
yum install vagrant_1.8.1_x86_64.rpm
{% endhighlight %}


下载virtualbox

{% highlight sh %} wget http://download.virtualbox.org/virtualbox/5.0.16/VirtualBox-5.0-5.0.16_105871_el6-1.x86_64.rpm 
yum install VirtualBox-5.0-5.0.16_105871_el6-1.x86_64.rpm
{% endhighlight %}

安装box

{% highlight sh %} 
vagrant init hansode/centos-6.6-x86_64; vagrant up --provider virtualbox 
{% endhighlight %}


