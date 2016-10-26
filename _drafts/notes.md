---
layout: page
title: notes
categories: notes
---


root运行vlc
  sed -i 's/geteuid/getppid/' /usr/bin/vlc
centos6 安装vlc
  yum install -y epel-release
  rpm -Uvh http://li.nux.ro/download/nux/dextop/el6/x86_64/nux-dextop-release-0-2.el6.nux.noarch.rpm
  yum install vlc

www.cutv.com/demo/live_test.swf


centos关闭selinux
  查看状态
    sestatus
  关闭
    sed -i 's/enforcing/disabled/g' /etc/selinux/config
  reboot

Strongswan
    yum install strongswan -y
    使用 PSK + XAUTH 形式连接 
        使用该方式 iOS 设备无需证书,只需要使用账户名,密码及密钥的情况下即可连接.
    配置 Strongswan
        编辑 /etc/ipsec.conf
    ./server_key.sh programmer-home.com
    ./client_key.sh bluewind bluewind1521@gmail.com
        buzhidaoba1
        wind buzhidaobahaha1