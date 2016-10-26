---
layout: "post"
title: "centos6 vncserver"
date: "2016-09-12 14:29"
---

安装tigervnc server
  yum install tigervnc-server

配置
  修改/etc/sysconfig/vncservers文件的以下内容
    VNCSERVERS="2:root"
    VNCSERVERARGS[2]="-geometry 1366x768 -nolisten tcp"
设置密码
  vncserver :2
启动
  service vncserver start

连接
  vncviewer x.x.x.x:2