---
layout: "post"
title: "centos6 haproxy"
date: "2016-09-13 15:27"
---

安装
  yum install -y haproxy

配置log
  yum install rsyslog -y
  编辑/etc/rsyslog.conf
    $ModLoad imudp
    $UDPServerRun 514
    # 绑定到本地地址
    $UDPServerAddress 127.0.0.1
  创建并添加以下内容到/etc/rsyslog.d/haproxy.conf
    local2.* /var/log/haproxy.log
    
    限定日志级别
    local2.=info     /var/log/haproxy-info.log
    local2.notice    /var/log/haproxy-allbutinfo.log
    
配置tcp转发

使用listen配置
  listen rtmp-proxy
        bind *:1935
        option tcplog
        mode tcp
        balance roundrobin
        server rtmp1 192.168.99.100:1955
        server rtmp2 192.168.99.100:1956
        
使用frontend与backend配置
  frontend rtmpserver *:1935
         mode tcp
         option tcplog
         default_backend rtmp
  
  backend rtmp
         mode tcp
         option tcplog
         balance roundrobin
         server 192.168.99.100:1955
         server 192.168.99.100:1956

开启统计,可以通过http://xxxxx:1936访问并查看当前的统计数据
  listen stats
    bind 172.17.230.194:1936
    mode http
    stats enable
    stats hide-version
    stats realm Haproxy\ Statistics
    stats uri /
    stats auth admin:password

启动
  service haproxy start
