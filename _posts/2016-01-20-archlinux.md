---
layout: post
title: archlinux
categories: linux archlinux
date: 2016-01-20 18:18:22 +0800
---

# pacman

* 升级

{% highlight sh %} pacman -Syu {% endhighlight %}

* 降级软件包

pacman安装或更新后，包会放在/var/cache/pacman/pkg/目录下，找到指定的包，然后执行pacman -U xxx

* 忽略软件包

{% highlight sh %} pacman -Syyu --ignore xxx,bbb,ddd {% endhighlight %}

# 字体设置及美化

* 首先安装infinality,具体安装方式参考wiki

	/etc/X11/xinit/xinitrc.d/infinality-settings.sh文件可以动态修改,该文件包含一些预定义的环境变量

* 安装字体

	可以直接将windows里的字体打包,然后进行安装.

	如果需要安装全局的字体,将字体文件移动到/usr/share/fonts目录.新增的字体文件必须为所有用户分配可读权限,使用chmod为所有文件分配0444,为目录分配0555.

	如果只为当前用户安装字体,直接将目录放到~/.local/share/fonts目录即可(~/.fonts目录已过期)

* 更新字体缓存

	fc-cache

# 声音

* 查看设备和驱动是否安装正确

{% highlight sh %}
lspci | grep -i audio
lsmod | grep snd_
{% endhighlight %}

* 安装alsa-utils

{% highlight sh %}
pacman -S alsa-utils
{% endhighlight %}

* 开启声音
alsa的所有channel默认都为静音,需要手动取消静音.

	1. 通过amixer取消静音

		amixer sset Master unmute

	2. 通过alsamixer取消静音
	
		alsamixer

		> channel下的MM表示静音,00表示非静音.使用键盘左右方向键移动,m取消静音