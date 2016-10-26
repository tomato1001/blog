---
layout: post
title: yum-download-dependencies
categories: ["yum download"]
date: 2015-10-10 14:02:57
---

* 首先安装`yum-utils`依赖
	
		yum install yum-utils

* 下载包与依赖

		repotrack sysstat -p /home/users/folder

* 安装下载的包及依赖

		rpm -Uvh *