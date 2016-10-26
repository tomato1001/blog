---
layout: page
title: rabbitmq
categories: rabbitmq

---

* 开启管理插件  
rabbitmq-plugins enable rabbitmq-management

* 查看所有插件
rabbitmq-plugins list

* "guest" user can only connect via localhost

	在/etc/rabbitmq/rabbitmq.config文件中加入
	[{rabbit, [{loopback_users, []}]}].
	然后即可登录management
