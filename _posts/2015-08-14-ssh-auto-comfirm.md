---
layout: post
title: ssh自动确认
categories: ['ssh']
date: 2015-08-14 10:07:10
---
通过ssh命令连接到主机时，如果指定的主机不存在于```~/.ssh/know_hosts```中时，ssh将询问是否加入主机到该文件中。当我们需要编写自动化脚本时，希望对该操作进行自动处理，可以通过以下方式进行连接

```sh
ssh -o "StrictHostKeyChecking no"
```
