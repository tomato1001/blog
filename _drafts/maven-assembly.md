---
layout: page.
title: maven-assembly
categories: maven assembly
date: 2016-03-22 14:35:41 +0800
---

首先在pom.xml中对assembly插件进行配置.descriptor用于指定assembly文件

{% highlight xml %}
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <descriptors>
            <descriptor>src/assembly/package.xml</descriptor>
        </descriptors>
        <finalName>${project.build.finalName}</finalName>
        <outputDirectory>dist</outputDirectory>
        <appendAssemblyId>false</appendAssemblyId>
    </configuration>
    <executions>
        <execution>
            <id>package-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

package.xml

{% highlight xml %}
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3 http://maven.apache.org/xsd/assembly-1.1.3.xsd">

    <id>ovsp-billing</id>
    <formats>
        <format>zip</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>
    <fileSets>
        <fileSet>
            <directory>src/assembly</directory>
            <includes>
                <include>*.txt</include>
            </includes>
            <outputDirectory></outputDirectory>
        </fileSet>
    </fileSets>
    <files>
        <file>
            <source>${project.build.directory}/${project.artifactId}.sql</source>
            <outputDirectory>stg</outputDirectory>
        </file>
        <file>
            <source>${project.build.directory}/${project.artifactId}.sql</source>
            <outputDirectory>prod</outputDirectory>
        </file>
        <file>
            <source>${project.build.directory}/${project.build.finalName}.${project.packaging}</source>
            <outputDirectory></outputDirectory>
        </file>
    </files>

</assembly>
{% endhighlight %}