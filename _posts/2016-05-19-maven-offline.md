---
layout: page
title: maven offline(离线模式)
categories: maven

---
在进行maven项目开发时，通常需要配置maven的资源库，以便于减少不必要的网络开销。当我们处在公司内部开发环境中时，可能会直接使用公司内部网络中的私有库，对项目依赖进行统一管理。但是，当网络无法访问私有库时，如：有时候为了赶项目，想在家里敲敲代码，如果你直接将项目复制，然后在家里编译时，你会发现项目根本无法进行编译。此时，我们需要maven offline，将项目的所有依赖离线成资源库，然后在pom.xml中指定离线的资源库.详细步骤如下:

* 将项目依赖存储到指定路径

    mvn dependency:go-offline -Dmaven.repo.local=/home/wind/job/c827/repo/

* 配置pom.xml将离线的仓库路径配置在第一位

{% highlight xml %}
<repositories>
    <repository>
        <id>commonDep</id>
        <name>common dependency</name>
        <url>file:///home/wind/job/c827/repo/</url>
    </repository>
    <!-- it used when collect dependencies before commit. If developer already download dependency, get it from local repo, save traffik and time -->
    <repository>
        <id>localPlugins</id>
        <name>local plugins</name>
        <url>file://${user.home}/.m2/repository</url>
    </repository>
</repositories>
<pluginRepositories>
    <pluginRepository>
        <id>commonDep</id>
        <name>common dependency</name>
        <url>file:///home/wind/job/c827/repo/</url>
    </pluginRepository>
    <!-- it used when collect dependencies before commit. If developer already download dependency, get it from local repo, save traffik and time -->       
    <pluginRepository>
        <id>localPlugins</id>
        <name>local plugins</name>
        <url>file://${user.home}/.m2/repository</url>
    </pluginRepository>
</pluginRepositories>
{% endhighlight %}