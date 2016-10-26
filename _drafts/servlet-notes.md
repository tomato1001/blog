---
layout: 
title: servlet-notes

---

# filter dispatcher
dispatcher用于限制filter的请求类型,默认为REQUEST.dispatcher分为REQUEST, FORWARD, INCLUDE和ERROR.

实例:

{% highlight xml %}
<filter-mapping>
  <filter-name>Logging Filter</filter-name>
  <url-pattern>/products/*</url-pattern>
  <dispatcher>REQUEST</dispatcher>
  <dispatcher>FORWARD</dispatcher>
</filter-mapping>
{% endhighlight %}

上面的filter将接受REQUEST和FORWARD类型的请求.

***注意: servlet2.4之前不支持此参数配置,且默认为所有类型***

# foward与redirect的区别

* forward

	转发到服务器内部存在的资源,所有操作在服务端内部完成

* redirect
	
	跳转到不同服务器或域名的资源,服务端返回一个header到浏览器,该header包含需要跳转的url,浏览器接收该header并发送新的请求到服务端