---
layout: post
title:  "Java ssl证书生成及tomcat集成"
date:   2015-05-13 22:38:00
categories: tomcat
---
### 根据本机IP生成认证
    keytool -genkey -keyalg RSA -alias tomcatsso -dname "CN=10.80.11.97" -storepass changeit

> * -storepass指定密码  
> * -alias指定别名

### 根据认证导出证书
	keytool -export -alias tomcatsso -file %java_home%/jre/lib/security/tomcatsso.crt" -storepass changeit

### 生成证书认证文件
	keytool -import -alias tomcatsso -file "%java_home%/jre/lib/security/tomcatsso.crt" -keystore "%java_home%/jre/lib/security/cacerts" -storepass changeit

> * 当执行时出现**java.io.IOException:keystore was tampered with,or password was incorrect**时，将security下的cacerts文件删除。（注意备份原始文件）

### 相关命令

* 从根证书中删除认证

		keytool -delete -alias tomcatsso -keystore "%JAVA_HOME%/jre/lib/security/cacerts" -storepass changeit  
		keytool -delete -alias tomcatsso -storepass changeit  

* 从列表中查询

		keytool -list -keystore "%JAVA_HOME%/jre/lib/security/cacerts" -storepass changeit

### 证书位置
在JDK  的 \jre\lib\security目录下能找到 “tomcatsso.crt”和” cacerts”，在C:\Documents and Settings\user目录下能找到 “.keystore”； 将“tomcatsso.crt”复制到JDK  的 \jre下，将“.keystore”复制到Tomcat的\conf 下***（可以是任意位置）***

### Tomcat设置
编辑Tomcat  conf下的server.xml文件，在connector的配置位置添加以下的配置：
添加https 协议，端口为 8443

{% highlight xml %}
<Connector port="8443" protocol="HTTP/1.1" 
SSLEnabled="true" maxHttpHeaderSize="8192"
		 	   keystorePass="changeit"   keystoreFile="conf/.keystore"
               maxThreads="150" minSpareThreads="25" maxSpareThreads="75"
               enableLookups="false" disableUploadTimeout="true"
               acceptCount="100" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS"/>

{% endhighlight %}

### 客户端导入证书
找到生成的证书文件tomcatsso.crt.执行如下命令：

	D:\Program_Files\Java\1.6.0_32\jdk\jre\bin\keytool.exe -import -trustcacerts -alias tomcat -file  
	D:\Program_Files\Java\1.6.0_32\jdk\jre\tomcat.crt -keystore  D:\Program_Files\Java\1.6.0_32\jdk/jre/lib/security/cacerts -storepass admin123

> * 当执行出现***java.io.IOException:keystore was tampered with,or password was incorrect***时，将security下的cacerts文件删除。（注意备份原始文件）  
> * 以上命令可解决***unable to find valid certification path to requested target***此类错误




