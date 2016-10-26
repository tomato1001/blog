---
layout: post
title: SSH密匙设置
categories: linux ssh-keys
date: 2015-06-24 10:02:14
---
# SSH密匙
ssh密匙(ssh keys)是一种比密码更加安全的ssh登录方式。ssh密匙一般由一对密匙组成：公匙(public)和私匙(private).生成密匙后，将公匙放入需要登录的服务器，客户端使用对应的私匙进行登录，当公匙与私匙匹配时，代表验证成功。在生成密匙时可以为密匙指定密码，指定密码后，使用证书登录时，将需要输入此密码。

## 创建ssh密匙
在客户端机器上创建密匙：

	ssh-keygen -t rsa

## 存储密匙及密码
当输入`ssh-keygen`命令后，将会询问一些问题:

	Enter file in which to save the key (/home/demo/.ssh/id_rsa):

私匙存储文件路径，默认为id_rsa.

	Enter passphrase (empty for no passphrase):

为密匙设置密码，默认为空代表不设置密码.

密匙生成的完整过程如下：

	ssh-keygen -t rsa
	Generating public/private rsa key pair.
	Enter file in which to save the key (/home/demo/.ssh/id_rsa): 
	Enter passphrase (empty for no passphrase): 
	Enter same passphrase again: 
	Your identification has been saved in /home/demo/.ssh/id_rsa.
	Your public key has been saved in /home/demo/.ssh/id_rsa.pub.
	The key fingerprint is:
	4a:dd:0a:c6:35:4e:3f:ed:27:38:8c:74:44:4d:93:67 demo@a
	The key's randomart image is:
	+--[ RSA 2048]----+
	|          .oo.   |
	|         .  o.E  |
	|        + .  o   |
	|     . = = .     |
	|      = S = .    |
	|     o + = +     |
	|      . o + o .  |
	|           . o   |
	|                 |
	+-----------------+

修改私匙权限

	chmod 600 ~/.ssh/id_rsa

因私匙比较重要，有些操作系统将强制私匙文件的权限必须只有创建者能够进行操作。一般建议
修改为`600`,有些建议设置为`400`

## 复制公匙
密匙生成后，需要将公匙加入到需要登录的服务器中.

可以通过`ssh-copy-id`命令将公匙添加到指定机器的`authorized_keys`文件中

	ssh-copy-id user@192.168.33.21

或者，直接使用命令行

	cat ~/.ssh/id_rsa.pub | ssh root@192.168.33.21 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"


## 使用密匙登录
当上面的设置完成后，我们就可以使用密匙进行登录了。在客户端机器(私匙所在机器)使用ssh命令登录：

	ssh root@192.168.33.21

以上命令将使用`~/.ssh/id_rsa`私匙文件.可以通过`-i`指定私匙文件：
	
	ssh -i ~/.ssh/id_rsa_custom root@192.168.33.21

如果登录时出现Authentication refused: bad ownership or modes for...错误,请确保服务器上对应用户目录下的.ssh目录及.ssh/authorized_keys文件的权限设置是否正确,一般.ssh目录权限为700,authorized_keys为600

	chmod 700 ~/.ssh
	chmod 600 ~/.ssh/authorized_keys

## 为root账号关闭ssh密码
当决定使用密匙登录后，以前的密码登录方式就可以关闭了。关闭之前请先确认是否可以通过密匙登录。

修改`/etc/ssh/sshd_config`：

	PermitRootLogin without-password #为root用户关闭密码登录
	PasswordAuthentication no #关闭密码验证

重新加载配置

	reload ssh


