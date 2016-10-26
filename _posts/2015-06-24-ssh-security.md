---
layout: page
title: ssh-security
categories: linux ssh-keys
date: 2015-06-24 11:16:05
---
# ssh安全设置
最近在查看服务器的ssh日志时，发现大量的`Failed password for root from...`纪录，看来服务器正处于被暴力破解中，重复猜测root密码。虽然猜中密码的可能性不大，但是总这么频繁的发送请求，对服务器网络肯定有影响，通过网上搜索后，主要有以下2种方法杜绝这类问题：

+ 修改ssh端口、禁止root登陆

	修改/etc/ssh/sshd_config

		Port 4848
		PermitRootLogin no

+ 禁用密码登录，使用RSA私匙登录（推荐采用此方法）
采用密匙登录的配置方法可以参考其它文章，下面只列出与安全相关的sshd设置

	修改/etc/ssh/sshd_config

		RSAAuthentication yes #RSA认证
		PubkeyAuthentication yes #开启公钥验证
		AuthorizedKeysFile .ssh/authorized_keys #验证文件路径
		PasswordAuthentication no #禁止密码认证
		PermitEmptyPasswords no #禁止空密码
		UsePAM no #禁用PAM

