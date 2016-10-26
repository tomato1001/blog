---
layout: page
title: linux-expect-ssh-automatic
categories: linux ssh-expect
date: 2015-10-12 16:26:04
---
* expect脚本

{% highlight sh %}
#!/usr/bin/expect -f
 
set ip [lindex $argv 0];
 
spawn ssh user@$ip
expect "password:"
send "master007\n"
interact
{% endhighlight %}

* expect嵌入到shell

{% highlight sh %}
#!/bin/sh

ip=$1
expect -c "
	spawn ssh user@$ip
	expect \"password:\"
	send \"master007\n\"
	interact
"
{% endhighlight %}

* expect结合rsync自动上传

{% highlight sh %}
#!/bin/sh

ip=$1
dir=$2

if [[ -z "$1" ]];then
        ip="172.17.230.136"
fi

if [[ -z "$2" ]];then
        dir="/home/wind/job/c813/GeneAnalysis/geneanalysis_server_release"
fi

expect -c "
        spawn bash -c \"rsync -r $dir/* root@$ip:/usr/local/arcvideo/gene\"
        expect \"password:\"
        send \"master007\n\"
        interact

{% endhighlight %}