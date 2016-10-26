---
layout: post
title: maven-compilation-error-duplicate-classes
categories: ["maven"]
date: 2015-10-20 10:16:51
---

在maven中使用maven-processor-plugin插件生成代码时，有时候会遇到compilation error duplicate classes导致无法编译，可以在maven-compiler-plugin插件中使用```-proc:none```配置解决该问题：

{% highlight xml %}
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>2.3.2</version>
    <configuration>
        <source>${java.version}</source>
        <target>${java.version}</target>
        <encoding>${encoding}</encoding>
        <compilerArgument>-proc:none</compilerArgument>
    </configuration>
</plugin>
{% endhighlight %}
