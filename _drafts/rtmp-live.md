---
layout: page
title: rtmp-live
categories: rtmp live
date: 2016-03-24 18:59:00 +0800
---

本文介绍通过nginx与nginx-rtmp实现rtmp服务器功能.

首先进入一个目录

cd /usr/build

克隆nginx-rtmp源码

git clone git://github.com/arut/nginx-rtmp-module.git

下载nginx源代码

wget http://nginx.org/download/nginx-1.8.1.tar.gz
tar xzf nginx-1.8.1.tar.gz
cd nginx-1.8.1

编译nginx与nginx-rtmp模块

./configure --add-module=/usr/build/nginx-rtmp-module
make
make install

配置直播流rtmp模式
要开启rtmp支持,需要在nginx.conf文件中增加rtmp协议配置.配置如下

{% highlight sh %}
worker_processes  1;

error_log  logs/error.log debug;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       8585;
        server_name  localhost;
        # rtmp stat
        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }

        location /stat.xsl {
            # the path of stat.xsl file on disk, you must have property permission of stat.xsl
            root /usr/local/nginx;
        }

        # rtmp control
        location /control {
            rtmp_control all;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
}



rtmp {
    server {
        listen 1935;
        ping 30s;
        notify_method get;

        application myapp {
            live on;
        }
    }
}


{% endhighlight %}

注意,上面的/stat.xsl配置中的root为该文件实际存在的目录,请保证nginx所属的用户有对应的权限可以操作该文件.如果需要查看文件模式及所属者,可以使用以下命令进行查看

{% highlight sh %}
namei -om xxxxxx
{% endhighlight %}

启动nginx

{% highlight sh %}
nginx -s start
{% endhighlight %}

发布直播流
nginx启动后,我们就可以将直播流数据发布城rtmp了.如上面配置,我们可以将flv的流发布到rtmp://127.0.0.1/myapp/xx,**xx**可以随意定义.

使用ffmpeg将mp4发布为rtmp

{% highlight sh %}
ffmpeg -re -i fffff.mp4 -c copy -f flv rtmp://127.0.0.1/myapp/111
{% endhighlight %}

执行后,可以通过ffplay进行播放

{% highlight sh %}
ffplay rtmp://127.0.0.1/myapp/111
{% endhighlight %}

使用flazr发布
flazr是一个java实现的rtmp服务器/客户端,用它也可以进行发布

{% highlight sh %}
./client.sh -live -host 127.0.0.1 -app myapp 111 ~/Downloads/xxxxx.flv
{% endhighlight %}



ffmpeg -re -i 鼠来宝4萌在囧途.BD1280超清中英双字.mp4 -c:v libx264 -c:a copy -b:v 500k -maxrate 500k -bufsize 500k -s 640x480 -profile high -deinterlace -f flv rtmp://114.55.57.154/live/slb
