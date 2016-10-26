---
layout: page
title: centos git server configuration
categories: centos git
date: 2016-02-18 09:47:09 +0800
---

首先在本机和服务器上安装git

> yum install git

然后在服务器上添加用户, 用于操作git

> useradd git
> passwd git

为了方便访问git,可以为git用户配置ssh无密码登录,首先在本机上创建ssh key

> ssh-keygen -t rsa

执行完成后,会在~/.ssh目录下生成id_rsa.pub(公钥)和id_rsa(私钥).将公钥复制到服务器

> cat ~/.ssh/id_rsa.pub | ssh git@remote-server "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"

操作完成后,登录到服务器并创建git项目目录

> mkdir -p ~/test/project-1.git

切换到创建的目录

> cd ~/test/project-1.git

创建空的repo

> git init --bare

repo创建完成后,我们需要在本地机器上创建repo.

> mkdir -p ~/git/project
> cd ~/git/project
> git init

添加文件到repo

> touch aa
> git add .

当每次进行了变更操作后,必须运行git add添加到修改记录中.

提交修改并指定对应的注释

> git commit -m "initialize files" -a

之前的操作都是在本地机器上进行的.下面将介绍如何提交到git服务器.
在提交到服务器之前,为了后续方便操作,先将git服务器的ssh连接信息写入到~/.ssh/config文件中

{% highlight sh %}
Host code-server
        HostName 11.22.33.44
        Port 22
        User git
        IdentityFile ~/.ssh/git_id_rsa
{% endhighlight %}

添加git服务器

> git remote add origin code-server:/home/git/test/project-1.git
>
> /home/git/test/project-1.git为git服务器上的实际项目路径

添加了git服务器后就可以进行提交操作了

> git push origin master

clone远程git资源库

> git clone code-server:/home/git/test/project-1.git

