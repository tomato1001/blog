---
layout: "post"
title: "arch linux notes"
date: "2016-06-13 13:17"
---

+ sysctl
sysctl会读取/etc/sysctl.d/*.conf和/usr/lib/sysctl.d/*.conf配置文件.当2个目录存在相同文件时,/usr/lib/sysctl.d下的文件会替换/etc/sysctl.d下的文件.