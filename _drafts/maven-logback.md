---
layout: post
title: maven-logback
categories: maven logback
date: 2016-03-21 15:38:59 +0800
---

添加以下依赖

{% highlight xml %}
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.1.5</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.1.5</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.16</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jcl-over-slf4j</artifactId>
    <version>1.7.16</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>log4j-over-slf4j</artifactId>
    <version>1.7.16</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jul-to-slf4j</artifactId>
    <version>1.7.16</version>
</dependency>
{% endhighlight %}

logback.xml

{% highlight xml %}
<configuration>
    <contextName>logback</contextName>
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!-- encoders are assigned the type
             ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
        <encoder>
            <pattern>%date [%thread] %-5level %C\(%line\) - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>log.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- daily rollover -->
            <fileNamePattern>log.%d{yyyy-MM-dd}.log</fileNamePattern>

            <!-- keep 30 days' worth of history -->
            <!--<maxHistory>30</maxHistory>-->
        </rollingPolicy>

        <encoder>
            <pattern>%date [%thread] %-5level %C\(%line\) - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="org.springframework" level="error"/>
    <logger name="com.aliyun.oss" level="error"/>


    <root level="info">
        <appender-ref ref="stdout" />
        <appender-ref ref="file" />
    </root>
</configuration>
{% endhighlight %}