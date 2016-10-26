---
layout: page
title: maven-findbugs
categories: maven findbugs
date: 2016-01-20 16:29:13 +0800
---

# 添加依赖

{% highlight xml %}
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>findbugs-maven-plugin</artifactId>
    <version>3.0.3</version>
</plugin>
{% endhighlight %}

# 执行

{% highlight sh %}
mvn clean compile
mvn findbugs:check
mvn findbugs:gui
{% endhighlight %}

