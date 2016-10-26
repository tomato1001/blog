---
layout: post
title: zookeeper
categories: zookeeper
date: 2015-12-31 14:29:38 +0800
---

# 1. 安装
下载对应的压缩包并解压.

# 2. 配置
conf目录下的文件是zookeeper的配置文件,可以对log4j,zookeeper及启动参数等进行配置.

初次安装后,conf下有个zoo_sample.cfg文件,该文件是一个简单的zookeeper配置文件,可以配置心跳,数据目录及客户端连接端口等.支持的参数可以参考[Configuration Parameters][sc_configuration].

因zookeeper启动时默认会读取conf/zoo.cfg文件,所以请确认conf目录下存在zoo.cfg文件.可以将zoo_sample.cfg复制并重命名为zoo.cfg.

## 2.1 jvm及系统变量设置
使用bin/zkServer.sh启动zookeeper时,会导入bin/zkEnv.sh中的变量,而zkEnv.sh中会导入conf/java.env.所以,通过java.env可以设置jvm参数或`-D`参数

### 指定nio实现
zookeeper默认的tcp通讯是基于java nio实现的,内部也提供netty实现,使用系统变量`zookeeper.serverCnxnFactory`即可指定具体的nio实现

	JVMFLAGS="-Dzookeeper.serverCnxnFactory=org.apache.zookeeper.server.NettyServerCnxnFactory"

## 2.2 Clustered (Multi-Server) 设置
在zoo.cfg文件中使用**server.id**进行设置.**id**对应zookeeper的myid文件中的值.当使用集群方式启动时,zookpper将读取数据目录下的myid文件获得当前服务器的id,该文件默认不存在,可以手动创建,如:`echo 1 > myid`

{% highlight sh %}
tickTime=2000
initLimit=5
syncLimit=2
dataDir=/root/temp
clientPort=2181
server.1=172.17.230.136:2888:3888
server.2=172.17.230.135:2888:3888
server.3=172.17.230.133:2888:3888
{% endhighlight %}

上面的配置中`server.1=172.17.230.136:2888:3888`,**1**对应myid文件中的值;**172.17.230.136**为zookeeper机器ip地址;**2888:3888**为端口配置,左边的端口用于连接到leader节点,右边的端口用于leader election.

[sc_configuration]: https://zookeeper.apache.org/doc/r3.4.7/zookeeperAdmin.html#sc_configuration

# 3. 启动
使用bin/zkServer.sh进行启动或停止操作

# 4. 管理

## 4.1 使用4个字符的命令
zookeeper支持通过一些命令来管理zookeeper,这些命令都是4个字符的英文,可以通过nc或telnent连接到zookeeper并发送这些命令,然后获取返回结果.详细的命令参考[Commands][zkCommands]

[zkCommands]: https://zookeeper.apache.org/doc/r3.4.7/zookeeperAdmin.html#sc_zkCommands

#### ruok
测试是否处于无错误运行状态.当返回**imok**时代表正常.无返回信息时代表非正常.

	echo ruok | nc 172.17.230.136 2181

#### stat
查看zookeeper状态以及连接的客户端

	echo stat | nc 172.17.230.136 2181