---
layout: "post"
title: "linode vps centos 7编译tcp_hybla模块"
date: "2016-06-12 15:36"
---
本文介绍如何在centos 7环境下配置tcp_hybla

tcp_hybla功能在各操作系统中默认是没有开启的, 需要通过手动加载模块并指定内核参数net.ipv4.tcp_congestion_control开启.受安装的内核影响, 开启tcp_hybla也不同. 当内核已包含tcp_hybla模块时, 只需要加载内核, 然后配置即可, 但是, 当没有tcp_hybla模块时, 需要手动编译tcp_hybla模块.以下为具体的配置步骤.


# 有tcp_hybla模块

加载模块

modprobe tcp_hybla

验证模块是否加载

lsmod | grep hybla

验证模块是否工作

sysctl net.ipv4.tcp_available_congestion_control

当输出结果中包含hybla,则代表已工作

将以下内容添加到/etc/sysctl.conf

net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.core.netdev_max_backlog = 250000
net.ipv4.tcp_mtu_probing=1
net.ipv4.tcp_congestion_control=hybla

然后执行sysctl -p使上面的配置生效

上面的步骤是通过手动加载的,当机器重启之后需要再次重新执行一遍,为了方便,可以在机器启动时自动加载tcp_hybla模块

在/etc/sysconfig/modules目录下添加hybla.modules文件并包含以下内容

#!/bin/sh

/sbin/modprobe tcp_hybla

然后为该文件设置执行权限

chmod +x hybla.modules

通过以上配置后, 每次重启后会自动加载模块


# 没有tcp_hybla模块, 手动编译tcp_hybla模块

+ 准备环境
更新系统并安装相关依赖
  # yum update && yum install ncurses-devel gcc make bc openssl-devel
查看当前内核版本
  # uname -r
  4.5.5-x86_64-linode69
  上面的4.5.5则为我们当前系统内核的版本
找到内核版本后,去linux内核官网下载对应的源代码包并解压
  # wget https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.5.5.tar.gz
  # tar -zxf linux-4.5.5.tar.gz

+ 编译模块
进入源码目录
  # cd linux-4.5.5
复制当前的内核配置
  # zcat /proc/config.gz > .config
配置内核参数
  # make menuconfig
  将Networking support -> Networking options -> TCP: advanced congestion control -> TCP-Hybla congestion ...设置为模块
编译
  # make prepare
  # make modules_prepare
  因我们只需要tcp_hybla模块,为了节省时间,指定只编译ipv4目录下的文件
  # make M=net/ipv4
删除debug符号(可选)
  strip --strip-debug net/ipv4/tcp_hybla.ko

+ 安装模块
  # mkdir -p /lib/modules/`uname -r`/extra
  # cp net/ipv4/tcp_hybla.ko /lib/modules/`uname -r`/extra
生成依赖
  depmod -a
  执行后上面的命令后,我们的模块会加入到/lib/modules/`uname -r`/modules.dep文件中,之后就可以使用modprobe加载模块了
加载模块
  modprobe tcp_hybla
  如果执行上述命令后出现如下错误
  modprobe: ERROR: could not insert 'tcp_hybla': Exec format error
  可以使用以下命令强制读取
  modprobe -f tcp_hybla

+ 其它

如果你编译的模块可以使用modprobe tcp_hybla加载成功,可以将tcp_hybla加入到/etc/modules-load.d目录下让系统自动加载该模块.如果需要使用-f参数进行强制加载才能成功加载模块,我们可以编写systemd service实现自动加载

echo '[Unit]
Description=Auto force load tcp_hybla module on boot

[Service]
Type=forking
ExecStart=/usr/sbin/modprobe -f tcp_hybla

[Install]
WantedBy=multi-user.target' > /etc/systemd/system/tcp-hybla.service

开机自动运行
systemctl enable tcp-hybla




