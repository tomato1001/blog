---
layout: post
title: linux时间与时区设置
categories: ['linux', 'timezone', 'date']
date: 2015-08-14 09:24:37
---
# 时间

+ 设置时间

		date -s "2015-01-01 12:20:01"

# 时区

+ 查看当前时区

		date +%z
		date +%Z

+ 设置时区

	1. 查找时区数据文件

		时区数据文件一般存放在```/usr/share/zoneinfo```目录下，比如：

			/usr/share/zoneinfo/Asia/Shanghai

	2. 修改```/etc/sysconfig/clock```设置新时区，修改此文件中的**ZONE**值为新的时区

	        ZONE="Asia/Shanghai"

	3. 创建符号链接

		为了使系统能够正确的找到设置的时区，需要建立一个时区文件的符号链接到```/etc/localtime```

			ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

	4. 重启机器使时区设置生效

			reboot

+ 通过**tzselect**设置时区

		tzselect