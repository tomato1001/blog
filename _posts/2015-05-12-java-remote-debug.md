---
layout: post
title:  "remote debug"
date:   2015-05-12 22:51:00
categories: java
---
使用以下参数进行remote debug  

``` sh
java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=8000 \
	-cp .:lib/* com.xxx.Application
``` 

``` sh
java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=8000 \
	-cp .:lib/* -Darcvideo.application.config=./conf/config.properties \
	-Dagent.config=./conf/agent.properties com.arcsoft.supervisor.agent.Application
```
运行之后，在ide中开启remote debug，指定对应的ip和端口（上面命令行指定的端口）
