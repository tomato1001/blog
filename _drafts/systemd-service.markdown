---
layout: "post"
title: "systemd service文件示例"
date: "2016-06-12 14:50"
categories: linux systemd
---

使用systemd方式配置ssserver命令

+ 创建service文件

touch /etc/systemd/system/ssserver.service

加入以下内容

{% highlight sh %}
[Unit]
Description=Shadowsocks server
After=network.target

[Service]
Type=forking
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks/config --fast-open -d start
ExecStop=/usr/bin/ssserver -c /etc/shadowsocks/config --fast-open -d stop
ExecReload=/usr/bin/ssserver -c /etc/shadowsocks/config --fast-open -d restart

[Install]
WantedBy=multi-user.target
{% endhighlight %}

+ 使用

{% highlight sh %}
# 重新加载service文件
systemctl daemo-reload
systemctl start ssserver
{% endhighlight %}
