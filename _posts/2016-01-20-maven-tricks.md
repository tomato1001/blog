---
layout: page 
title: maven-tricks
categories: maven tricks
date: 2016-01-20 16:21:47 +0800
---

# 故障排查命令

* -X开启debug

	mvn clean install –X

* 查看依赖树

	mvn dependency:tree

* 查看环境变量以及java属性

	mvn help:system

* 查看最终的pom信息

	mvn help:effective-pom

* 查看classpath依赖

	mvn dependency:build-classpath

* 分析依赖

	mvn dependency:analyze

# 自定义archetype

* 首先创建一个空的maven项目
* 如果之前用开发工具建立的项目，需要将相关的文件删除，否则也会打包进最后的archetype中.
比如:intellij idea创建的工程,需要将.idea删除；eclipse创建的工程,需要将eclipse相关的文件删除
* 然后执行mvn archetype:create-from-project，执行后，编译的文件在target/generated-sources/archetype目录下
* 切换到target/generated-sources/archetype目录并执行mvn install
* 使用archetype

{% highlight bash %}
mvn archetype:generate -DarchetypeGroupId=<archetype-groupId> -DarchetypeArtifactId=<archetype-artifactId> -DarchetypeVersion=<archetype-version> -DgroupId=<my.groupid> -DartifactId=<my-artifactId>
{% endhighlight %}

# 自动修改version
当项目包含多个子模块，而且不想用release来发布时，可以通过versions插件来自动设置版本

* 设置为指定的版本：***mvn versions:set -DnewVersion=1.0.31***
执行后会在每个模块目录下多出一个文件，该文件为对之前版本的备份。此时，执行commit将会删除备份文件或者执行revert将还原到之前的修改

* mvn versions:commit
* mvn versions:revert

# 下载源代码，依赖及docs

* mvn dependency:sources
* mvn dependency:resolve -Dclassifier=Javadoc

# 自定义war包构建方式

{% highlight xml %}
<plugin>
    <artifactId>maven-war-plugin</artifactId>
    <groupId>org.apache.maven.plugins</groupId>
    <version>2.3</version>
    <configuration>
        <warName>${project.artifactId}</warName>
        <outputDirectory>${project.parent.basedir}/build/web</outputDirectory>
    </configuration>
</plugin>
{% endhighlight %}
configuration参数说明

> warName修改war包名称  
outputDirectory修改war包存放路径

# struts2 archetype
用于创建struts2项目

{% highlight sh %}
mvn archetype:generate -B \
                         -DgroupId=com.phome.samples \
                         -DartifactId=struts2 \
                         -DarchetypeGroupId=org.apache.struts \
                         -DarchetypeArtifactId=struts2-archetype-starter \
                         -DarchetypeVersion=2.3.24.1
{% endhighlight %}

# 打包成可执行jar包并将依赖也打入jar包中
将所有的class及依赖打包成单个可执行的jar包.注意:此打包方式在打包spring开发的程序时会导致问题

{% highlight xml %}
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <archive>
            <manifest>
                <mainClass>com.phome.samples.watchfolder.FileMonitor</mainClass>
            </manifest>
        </archive>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
</plugin>
{% endhighlight %}

# 将generated-sources添加到class path

{% highlight xml %}
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>build-helper-maven-plugin</artifactId>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>add-source</goal>
            </goals>
            <configuration>
                <sources>
                    <source>target/generated-sources</source>
                </sources>
            </configuration>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

# 将jar包添加到本地maven库

{% highlight sh %}
mvn install:install-file -Dfile=<path-to-file> -DgroupId=<group-id> \
    -DartifactId=<artifact-id> -Dversion=<version> -Dpackaging=<packaging>
{% endhighlight %}

# 使用shade打包成可执行jar包并将依赖也打入jar包中
{% highlight xml %}
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <configuration>
        <finalName>quartz-cluster-test</finalName>
        <outputDirectory>${project.basedir}/build</outputDirectory>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.phome.samples.quartz.App</mainClass>
                    </transformer>
                    <!-- 添加spring.handlers -->
                    <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                        <resource>META-INF/spring.handlers</resource>
                    </transformer>
                    <!-- 添加spring.schemas -->
                    <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                        <resource>META-INF/spring.schemas</resource>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
{% endhighlight %}