---
layout: page
title: logback-mybatis
categories: logback mybatis
date: 2016-01-20 17:00:57 +0800
---
# logback mybatis开启sql日志

+ 首先在sqlMapconfig.xml中加入

{% highlight xml %}
<setting name="logPrefix" value="dao."/>
{% endhighlight %}


+ 在logback.xml中加入

{% highlight xml %}
<logger name="dao" level="DEBUG">
    <appender-ref ref="STDOUT" />
</logger>
{% endhighlight %}

mybatis打印sql语句的log是与sql id对应的,如:某模块的插入sql id为com.ff.aa.insert, 直接在logback中配置如下也可以打印sql

{% highlight xml %}
<logger name="com.ff.aa" level="DEBUG"/>
{% endhighlight %}

之前配置前缀的方式实际上是mybatis在所有的sql id前面增加了**dao.**,所以在logback中我们只需要指定开始的包名就能实现打印所有模块的sql.