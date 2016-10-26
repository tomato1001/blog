---
title: atom install
layout: post
---

* 安装nodejs和npm

* 下载文件并解压

{% highlight sh %}
mkdir /tmp/atom-editor
wget https://github.com/atom/atom/archive/v1.7.3.zip
unzip v1.7.3.zip
cd atom-1.7.4
{% endhighlight %}

* 环境配置

因npm官网速度比较慢，可以直接使用国内taobao的镜像。手动将https://registry.npm.taobao.org/-/all返回的内容存储到~/.npm/registry.npm.taobao.org/-/all/.cache.json。然后执行如下命令

{% highlight sh %}
npm config set registry https://registry.npm.taobao.org --永久生效
{% endhighlight %}

设置apm代理用于下载atom的package，将以下内容加入到~/.atom/.apmrc中

{% highlight sh %}
http-proxy=http://xxxxx
https-proxy=http://xxxxx
proxy=http://xxxxx
strict-ssl=false
{% endhighlight %}

* 编译

{% highlight sh %}
script/build
{% endhighlight %}

* 安装

{% highlight sh %}
sudo script/grunt install
{% endhighlight %}
