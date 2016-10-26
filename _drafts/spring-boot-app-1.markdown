---
layout: "post"
title: "spring boot app 1"
date: "2016-06-22 14:17"
---

首先进行项目搭建,可以直接访问http://start.spring.io/进行可视化的项目配置,完成后将直接生成项目结构文件.也可以直接使用下面的pom配置.

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>aa.bb.cc</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1</version>
	<packaging>jar</packaging>
	<name>demo</name>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.3.5.RELEASE</version>
		<relativePath/>
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<java.version>1.7</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
	
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>

# application.properties相关设置
该文件是spring boot的核心文件,所有功能都能通过此配置文件进行开启,关闭或调整.

*　freemarker设置

spring.freemarker.enabled=true
spring.freemarker.cache=false
spring.freemarker.charset=UTF-8
spring.freemarker.suffix=.ftl
spring.freemarker.template-loader-path=file:${user.dir}/templates/,classpath:/templates/

# 优先读取本地路径的模板
spring.freemarker.prefer-file-system-access=true

* i18n
默认已经开启了i18n, 以下配置项可以设置资源消息文件
spring.messages.basename=messages/message

* server
spring.main.banner_mode=off
spring.profiles.active=${profiles:dev}
server.port=${port:8085}

# logger
logging.file=${logging-file:${user.dir}/logs/log.log}

* logback
在resources下新建logback-spring.xml文件,加入以下内容

<configuration>
    <contextName>logback</contextName>
    <springProperty scope="context" name="logFile" source="logging.file"/>
    <springProfile name="dev">
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
    </springProfile>

    <springProfile name="prod">
        <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${logFile}</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <!-- daily rollover -->
                <fileNamePattern>${logFile}-%d{yyyy-MM-dd}.%i</fileNamePattern>
                <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                    <maxFileSize>50MB</maxFileSize>
                </timeBasedFileNamingAndTriggeringPolicy>
            </rollingPolicy>

            <encoder>
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
    </springProfile>

    <root level="info">
        <springProfile name="dev">
            <appender-ref ref="STDOUT"/>
        </springProfile>
        <springProfile name="prod">
            <appender-ref ref="FILE"/>
        </springProfile>
    </root>
</configuration>

# 自定义freemarker

{% highlight java %} 
@Configuration
public class FreeMarkerAutoConfiguration extends org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration {


    @Configuration
    public static class FreeMarkerNonWebConfiguration extends FreeMarkerConfiguration {

        @Bean(name = "freemarker")
        public FreeMarkerConfigurationFactoryBean freeMarkerConfiguration() {
            FreeMarkerConfigurationFactoryBean freeMarkerFactoryBean = new FreeMarkerConfigurationFactoryBean();
            applyProperties(freeMarkerFactoryBean);
            return freeMarkerFactoryBean;
        }

    }
}
{% endhighlight %}