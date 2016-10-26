---
layout: post
title:  "maven profile使用"
date:   2015-05-12 23:10:00
categories: maven
---
在开发过程中，我们经常需要根据不同的环境读取不同的配置文件。比如日志、数据库等配置文件，而根据环境的不同，我们需要设置的参数也将不同：

* 在开发环境中，日志一般输出到终端，这样比较容易定位问题，数据库连接可能也是指向到自己开发机器上的数据库。  
* 在产品环境中，日志将会输出到指定的日志文件，数据库配置的参数也与开发环境不同。  
通常我们的配置文件的名称都是固定的，比如log4j.properties、database.properties，所以要解决以上的问题，可以采用最原始的解决方法：
* 当切换到不同的环境之前，手动修改这些配置文件的参数设置项
但是，以上的解决方法非常不灵活，每次环境切换都需要手动修改，而且非常容易出错。那么，是否有更好的方法？构建时自动选择指定环境的配置文件？答案是肯定的，那就是maven的profile。
具体实现方式如下：
	* 建立多份配置文件。
	* 为每个环境配置profile
	* 构建时指定profile，使用-P可以指定需要使用的profile。如：mvn -Ptest

下面是对应的项目结构:

<div style="width: 200px;text-align: center;position: relative; left: 230px;">
	<img src="{{'/images/maven-profile-1.png' | prepend: site.baseurl}}" alt="">
</div>
<br>
可以通过2种方式达到我们的目的，以下为详细配置：

* 通过antrun插件实现，antrun插件可以在maven中使用ant命令

{% highlight xml %}
<profiles>
    <profile>
        <id>test</id>
        <build>
            <plugins>
                <plugin>
                    <artifactId>maven-antrun-plugin</artifactId>
                    <executions>
                        <execution>
                            <phase>test</phase>
                            <goals>
                                <goal>run</goal>
                            </goals>
                            <configuration>
                                <tasks>
                                    <copy todir="${project.build.outputDirectory}" overwrite="true">
                                        <fileset dir="${project.basedir}/profiles/test">
                                            <include name="*"/>
                                        </fileset>
                                    </copy>
                                </tasks>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
    <profile>
        <id>production</id>
        <build>
            <plugins>
                <plugin>
                    <artifactId>maven-antrun-plugin</artifactId>
                    <executions>
                        <execution>
                            <phase>test</phase>
                            <goals>
                                <goal>run</goal>
                            </goals>
                            <configuration>
                                <tasks>
                                    <copy todir="${project.build.outputDirectory}" overwrite="true">
                                        <fileset dir="${project.basedir}/profiles/production">
                                            <include name="*"/>
                                        </fileset>
                                    </copy>
                                </tasks>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
{% endhighlight %}

* 通过resources插件实现  

{% highlight xml %}
<profiles>
    <profile>
        <id>test</id>
        <build>
            <plugins>
                <plugin>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>2.6</version>
                    <executions>
                        <execution>
                            <id>production-copy-resources</id>
                            <phase>process-resources</phase>
                            <goals>
                                <goal>copy-resources</goal>
                            </goals>
                            <configuration>
                                <overwrite>true</overwrite>
                                <outputDirectory>${project.build.outputDirectory}</outputDirectory>
                                <resources>
                                    <resource>
                                        <directory>${project.basedir}/profiles/test</directory>
                                    </resource>
                                </resources>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
{% endhighlight %}

配置完成后，可以通过指定profile来实现切换，如：

* mvn -Ptest package  --使用profiles/test下的配置文件
* mvn -Pproduction package --使用profiles/production下的配置文件
* mvn package --使用默认的配置文件src/main/resources

