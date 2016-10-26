---
layout: "post"
title: "centos-notes"
date: "2016-06-22 16:52"
---

+ centos删除内核

安装yum-utils
  yum install yum-utils
查看已安装的内核
  rpm -qa | grep kernel
删除内核,以下命令只保留最新的2个内核
  package-cleanup --oldkernels --count=2

+ yum groupinstall安装之后无法重新安装解决方式
  yum groups mark install "Development Tools"
  yum groups mark convert "Development Tools"
  yum groupinstall "Development Tools"