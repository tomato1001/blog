---
layout: page
title: maven-clean
categories: maven clean-plugin
date: 2016-03-22 14:38:39 +0800
---

maven-clean-plugin可以用于清除自定义的资源

{% highlight xml %}
<plugin>
    <artifactId>maven-clean-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <filesets>
            <fileset>
                <directory>dist</directory>
                <includes>
                    <include>*</include>
                </includes>
            </fileset>
        </filesets>
    </configuration>
</plugin>
{% endhighlight %}