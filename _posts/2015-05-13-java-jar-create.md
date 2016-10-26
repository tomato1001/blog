---
layout: post
title:  "java可执行jar包制作"
date:   2015-05-13 22:18:00
categories: java
---
* 访问jar包中的图片资源： 

{% highlight java %}
Image image = Toolkit.getDefaultToolkit().getImage(this.getClass().getResource("/tool.png"));
{% endhighlight %}

* 先使用jar cvf pnote.jar –C bin/ . 将编译后的字节码制作成jar包
* 修改jar包中的MANIFEST.MF文件，增加以下内容:  

        Class-Path: ./lib/sqlite4java.jar ./lib/sqlite4java-win32-x86.dll
        Main-Class: org.pnote.Pnote

