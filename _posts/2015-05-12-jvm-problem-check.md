---
layout: post
title:  "jvm故障排查技巧"
date:   2015-05-12 18:33:00
categories: java tuning
---
* jinfo  
	* jinfo -flags --查看默认参数  
	* jinfo -flag --动态设置参数，当managerable开启时才能设置  
* jmap  
	* 不要在生产环境中使用，可能会导致jvm挂起
* gcore
	* 可以生成core dump，最好的保留现场的方式，生成后的dump可以直接用jstack、jmap或者eclipse memory analyze分析
	* 需要先尝试，有些jvm不支持，可能无法获得dump
*	可以使用google perftools分析堆外内存
* ulimit的各个限值也比较重要
* blktrace+debugfs+btrace排查cpu或io