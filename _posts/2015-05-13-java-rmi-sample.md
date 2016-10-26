---
layout: post
title:  "java rmi sample"
date:   2015-05-13 22:52:00
categories: java
---
### 定义远程接口

{% highlight java %}
package server;

import java.rmi.Remote;
import java.rmi.RemoteException;

public interface Hello extends Remote {
    
    String sayHello(String name) throws RemoteException;

}
{% endhighlight %}
> 远程方法必须抛出RemoteException

### 服务实现

{% highlight java %}
package server;

import java.rmi.RemoteException;

public class Server implements Hello {

    @Override
    public String sayHello(String name) throws RemoteException {
	return "say hello to " + name;
    }
    
}
{% endhighlight %}

### 服务启动类

{% highlight java %}
package client;

import java.rmi.NotBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

import server.Hello;

public class Client {

    public static void main(String[] args) {
	try {
	    Registry registry = LocateRegistry.getRegistry(args[0]);
	    Hello hello = (Hello) registry.lookup("hello");
	    String res = hello.sayHello(args[1]);
	    System.out.println(res);
	}catch (RemoteException e) {
	    e.printStackTrace();
	}catch (NotBoundException e) {
	    e.printStackTrace();
	}

    }

}
{% endhighlight %}

### 客户端实现

{% highlight java %}
package client;

import java.rmi.NotBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

import server.Hello;

public class Client {

    public static void main(String[] args) {
	try {
	    Registry registry = LocateRegistry.getRegistry(args[0]);
	    Hello hello = (Hello) registry.lookup("hello");
	    String res = hello.sayHello(args[1]);
	    System.out.println(res);
	}catch (RemoteException e) {
	    e.printStackTrace();
	}catch (NotBoundException e) {
	    e.printStackTrace();
	}

    }

}
{% endhighlight %}

### 编译文件
使用javac或ide编译文件

### 开始java rmi registry、server and client

> * 运行JAVA_HOME里bin目录下的rmiregisty，默认端口为1099，如果需要改变端口，可在命令后加入端口号：  
	* Windows：start rmiregistry  
	* Linux：rmiregistry &  
	* rmiregistry 2011 –绑定到2011端口  

### 运行服务启动类、客户端进行测试
