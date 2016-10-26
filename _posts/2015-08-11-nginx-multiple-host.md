---
layout: post
title: nginx多主机配置
categories: ['nginx']
date: 2015-08-11 16:36:40
---
# nginx多主机配置

多主机即将多个不同的域名指向到同一个服务器。比如：a.com和b.com，这2个网站的功能完全不同，但是，为了节省资源，我们希望把这2个网站放在同一台服务器。当我们使用nginx作为web服务器时，通过虚拟主机功能，对每个站点进行单独配置，可以很方便的达到我们的目标。

## 检查nginx.conf文件

检查该文件中的```include /etc/nginx/conf.d/*.conf;```,确保该行没有被注释。

## 配置站点

在/etc/nginx/conf.d目录下分别新建如下2个文件，文件名称没有限定，可以随意命名

### site1.conf

```sh
server {
    listen       80;
    server_name  *.a.com;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```

### site2.conf

```sh
server {
    listen       80;
    server_name  *.b.com;

    location / {
        root   /usr/share/nginx/html2;
        index  index.html index.htm;
    }
}
```

通过上述配置之后，当访问a.com和b.com时，nginx将根据访问的域名将请求交给对应的虚拟主机进行处理。