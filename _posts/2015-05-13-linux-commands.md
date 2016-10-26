---
layout: post
title:  "Linux commands and tricks"
date:   2015-05-13 23:19:00
categories: linux
---
### centos下运行history时,没有显示执行命令的时间,解决方案为：
* 添加以下环境变量到/etc/profile中
{% highlight sh %}
export HISTTIMEFORMAT="%F %T `whoami` "
{% endhighlight %}

* 使环境变量生效
{% highlight sh %}
source /etc/profile
{% endhighlight %}
		

### 系统crash时,可以通过crash命令查看具体原因(xxx为内核版本)
* 需要安装kernel-debug-xxx和kernel-debuginfo-xxx
* 系统crash后会在/var/crash目录下生成vmcore文件,通过crash命令分析此文件
{% highlight sh %}
crash /usr/lib/debug/lib/modules/xxx/vmlinux /var/crash/127.0.0.1-xxx/vmcore
{% endhighlight %}

### linux下实现单网卡双ip
{% highlight sh %}
ip addr add 192.168.56.103/24 dev eth0
{% endhighlight %}
> ip地址后的24为子网掩码位数

### 添加默认路由
{% highlight sh %}
route add default gw 192.168.1.1
{% endhighlight %}

### 启动时添加默认路由及ip等信息（也可解决重启后默认路由丢失情况）
	修改/etc/sysconfig/network-scripts/ 目录下的ifcfg-ethX信息

### convert man to pdf
{% highlight sh %}
man -t man | ps2pdf -> man.pdf
{% endhighlight %}

### cpu info
总核数 = 物理CPU个数 X 每颗物理CPU的核数 

总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 2(超线程数)

* 查看cpu型号

{% highlight sh %}
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
{% endhighlight %}
	
8  Intel(R) Xeon(R) CPU E3-1275 v3 @ 3.50GHz (有8个逻辑CPU)

* 查看物理CPU个数(主板上有几个CPU)

{% highlight sh %}
cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
{% endhighlight %}

* 查看逻辑CPU的个数(逻辑核)

{% highlight sh %}
cat /proc/cpuinfo| grep "processor"| wc -l
{% endhighlight %}

* 查看每个物理CPU中core的个数(物理核)

{% highlight sh %}
cat /proc/cpuinfo| grep "cpu cores"| uniq
{% endhighlight %}

### Cleanup tmp directory
{% highlight sh %}
find /tmp -mtime +7 -and -not -exec fuser -s {} ';' -and -exec echo {} ';'
{% endhighlight %}

### 查看从172.17.193.144发送过来的udp包
{% highlight sh %}
tcpdump -i eth1 udp and src net 172.17.193.144
tcpdump -i eth1 ip multicast and src net 172.17.193.144 --抓取组播
tcpdump -i eth1 udp and dst port 8901    --抓取8901端口收到的udp
{% endhighlight %}

### 查看连接数
{% highlight sh %}
ss -s
{% endhighlight %}
	
### 查看网卡流量
{% highlight sh %}
iftop tcptrack
{% endhighlight %}
	
### 查看端口使用
{% highlight sh %}
	lsof -i :portNumber
	lsof -i tcp:portNumber
	lsof -i udp:portNumber
	lsof -i :80 | grep LISTEN 
{% endhighlight %}

### 组播地址为224.0.0.0对应的子网掩码为240.0.0.0

### 使用iostat查看io性能
{% highlight sh %}
iostat -x -m 1 10
{% endhighlight %}
	
### 使用dd测试磁盘性能
{% highlight sh %}
dd if=/dev/zero of=/dddd bs=1G count=10 oflag=direct --oflag=direct 将直接写入硬盘
{% endhighlight %}

### ifconfig
+ 设置IP和子网掩码
{% highlight sh %}
ifconfig eth0 192.168.5.40 netmask 255.255.255.0
{% endhighlight %}
+ 设置网关
{% highlight sh %}
route add default gw 192.168.5.1
{% endhighlight %}

# 查看发行版信息
{% highlight sh %}
cat /etc/issue
{% endhighlight %}

# 硬盘

* 查看块设备
{% highlight sh %}
lsblk
{% endhighlight %}

* 是否ssd
{% highlight sh %}
cat /sys/block/sda/queue/rotational
{% endhighlight %}
输出1代表hdd，0为ssd

# 重新mount为可读写设备
{% highlight sh %}
mount /dev/sdc1 -o rw,remount /xxx/fff
{% endhighlight %}

# linux mount mac共享目录
{% highlight sh %} 
mount.cifs //172.28.100.173/Test /mnt/data/xxxxx -o user=share,password=1111,nounix,sec=ntlmssp,noperm,rw
{% endhighlight %}

# 清除内存缓存
{% highlight sh %}
echo 3 > /proc/sys/vm/drop_caches
{% endhighlight %}

# 关闭ipv6
{% highlight sh %}
echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6  echo 1 /proc/sys/net/ipv6/conf/default/disable_ipv6
{% endhighlight %}

# 删除修改时间为2小时之前的文件
{% highlight sh %}
find . -mmin +120 -type f -name "*.ts" -exec rm -rf {} \;
{% endhighlight %}

# 尺寸最大的文件降序
{% highlight sh %}
du -h | sort -h -r 
{% endhighlight %}

# 查看文件夹占用大小
	{% highlight sh %} du -sh {% endhighlight %}

# 查看子文件夹占用大小
	{% highlight sh %} du -h --max-depth=1 | sort -hr {% endhighlight %}

# netstat

* 统计连接state

{% highlight sh %}
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
{% endhighlight %}

# sed

替换
sed -i 's/aa/bb/g' xx.file

使用shell变量
sed -i "s/aa/$var/g" xx.file

当变量中包含特殊字符,如/时
sed -i "s#aa#$var#g" xx.file

