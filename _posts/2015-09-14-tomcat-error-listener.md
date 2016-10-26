---
layout: post
title: tomcat Error listenerStart
categories: tomcat error-listenerStart
date: 2015-09-14 13:59:38
---
使用tomcat时，有时候会出现```Error listenerStart```，但是tomcat和程序的日志文件中都没有异常信息，且应用无法访问。当出现这种情况时，可以通过打开tomcat的日志，让tomcat输出具体的异常信息。

* 在WEB-INF/classes目录下新建一个名称为```logging.properties```的文件，并加入以下内容：

		org.apache.catalina.core.ContainerBase.[Catalina].level = INFO
		org.apache.catalina.core.ContainerBase.[Catalina].handlers = java.util.logging.ConsoleHandler
		
* 重新启动tomcat，在tomcat/logs/catalina.out文件中查看是否有异常信息