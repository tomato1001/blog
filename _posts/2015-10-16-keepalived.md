---
layout: post
title: keepalived
categories: linux keepalived
date: 2015-10-16 10:09:45
---

keepalived默认将根据优先级与状态自动进行切换，如果主机挂了，备机开始接管，此时主机恢复将会从备机又切换回主机，这样将导致多余的切换且对应用系统有影响。可以通过以下2种方式解决该问题

+ 将状态都设置为BACKUP，并保证主机的优先级高于备机的优先级且在主机的配置中加入nopreempt

主机配置

{% highlight sh %}
! Configuration File for keepalived

vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 251
    priority 150
    nopreempt
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass password123
    }
    virtual_ipaddress {
        10.0.0.200
    }
}
{% endhighlight %}

备机配置

{% highlight sh %}
! Configuration File for keepalived

vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 251
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass password123
    }
    virtual_ipaddress {
        10.0.0.200
    }
}
{% endhighlight %}

+ 主机与备机都不设置state并将优先级设置为相同

主机配置

{% highlight sh %}
! Configuration File for keepalived

vrrp_instance VI_1 {
    interface eth1
    virtual_router_id 251
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass password123
    }
    virtual_ipaddress {
        10.0.0.200
    }
}
{% endhighlight %}

备机配置

{% highlight sh %}
! Configuration File for keepalived

vrrp_instance VI_1 {
    interface eth1
    virtual_router_id 251
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass password123
    }
    virtual_ipaddress {
        10.0.0.200
    }
}
{% endhighlight %}

# 使用单播替代组播作为vrrp传输方式

主机配置

{% highlight sh %}
! Configuration File for keepalived

vrrp_instance VI_1 {
    interface eth1
    virtual_router_id 251
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass password123
    }
    unicast_peer {
        10.0.0.100  # 加入其它机器的地址
    }
    virtual_ipaddress {
        10.0.0.200
    }
}

{% endhighlight %}

备机配置

{% highlight sh %}
! Configuration File for keepalived

vrrp_instance VI_1 {
    interface eth1
    virtual_router_id 251
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass password123
    }
    unicast_peer {
        10.0.0.101
    }
    virtual_ipaddress {
        10.0.0.200
    }
}

{% endhighlight %}