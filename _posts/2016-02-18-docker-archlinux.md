---
layout: post
title: docker on archlinux
categories: docker archlinux
---

# 安装docker

```sh
sudo pacman -S docker
```

# 启动docker

启动docker服务

```sh
systemctl start docker
```

如果需要开机启动,则执行以下命令

```sh
systemctl enable docker
```

# 添加当前用户到docker组

如果需要在非root用户下运行docker则需要将用户加入到docker组

```sh
gpasswd -a yourusername docker
```

重新登录或使用以下命令使作用到当前session

```sh
newgrp docker
```

# 修改docker0的ip地址
编辑/usr/lib/systemd/system/docker.service文件,在ExecStart这一行最后加入--bip=172.18.1.1/24,然后重启docker

# 手动构建docker image
因直接从docker hub中pull太慢,我们可以找到对应的image的github地址,然后将Dockerfile和image文件(一般为.tar.gz文件)下载到本地,然后使用docker进行build

下载image文件
wget https://raw.githubusercontent.com/CentOS/sig-cloud-instance-images/98bda021f98ad46991afcd9f8ca657bce762e631/docker/centos-6-docker.tar.xz

下载Dockerfile
wget https://raw.githubusercontent.com/CentOS/sig-cloud-instance-images/98bda021f98ad46991afcd9f8ca657bce762e631/docker/Dockerfile


```sh
docker build -t imagename .
```
上面的imagename对应自定义的名称

# 运行

执行命令

```sh
docker run imagename /bin/echo "Hello world"
```

交互式运行

```sh
docker run -t -i imagename /bin/bash
```

查看所有镜像

docker images

从运行的容器创建镜像

docker commit <container-id> <image-name>

docker ps 列出运行的容器
docker ps -a 列出所有容器
docker rm <container-id> 删除容器, 多个用空格分隔
docker rmi <image-name> 删除镜像

将主机当前目录挂载到/opt/data
docker run -v $PWD:/opt/data -it centos6 /bin/bash
路径必须是绝对路径

docker export <CONTAINER ID> > /home/export.tar 导出容器

指定端口映射
docker run -d -p 1935:1935 centos6:srs-1

# 设置host
docker run -h srs1.test.com -d -v /dat:/opt/data -it centos6:srs-1

docker网络设置
  在/etc/sysconfig/docker文件中配置DOCKER_OPTS环境变量.如:
    other_args="--bip=10.0.3.1/16"
  自定义br
    停止docker并删除br配置
      service docker stop
      ip link set dev docker0 down
      brctl delbr docker0
      iptables -t nat -F POSTROUTING
    创建bridge
      brctl addbr br0
      ip addr add 10.0.33.1/24 dev br0
      ip link set dev br0 up
    写入配置文件
    echo 'other_args="-b=br0"' >> /etc/sysconfig/docker
    service docker start
    
    Confirming new outgoing NAT masquerade is set up
      iptables -t nat -L -n
      
      

Dockerfile

  FROM centos6:base
  MAINTAINER zw3797

  COPY supervisord.conf /etc/supervisord.conf


  ENV SRS_AGENT_VERSION 0.0.3
  WORKDIR /usr/local/arcvideo/srs-agent
  ADD ovsp-srs-agent-0.0.3.zip ./
  RUN unzip ovsp-srs-agent-$SRS_AGENT_VERSION.zip -d ./
  COPY app.properties app.properties

  CMD ["/usr/bin/supervisord", "-n"]

docker run -d -h srs1-origin -e APPNAME=srs-forwarder-1 --name=f1 srs-forward
docker run -d -h srs2-origin -e APPNAME=srs-forwarder-2 --name=f2 srs-forward

docker run -d -h srs1-node -e APPNAME=srs-node-1 --link=f1 --link=f2 --name=n1 srs-node
docker run -d -h srs1-node -e APPNAME=srs-node-2 --link=f1 --link=f2 --name=n2 srs-node

docker run -d -h srs-cluster-1 -e APPNAME=srs-cluster-1 -p 9904:9904 --link=mysql --name=s1 srs-cluster

docker run -d -h mysql-server --name=mysql mysql:base

导出image
docker save <image name> > xxx.tar

# 引用到其它container的host
docker run -d -h srs1-node -e APPNAME=srs-node-1 --link a4d1f3f92547 srs-node

# 使用shell脚本启动程序时需要使用exec,否则将无法接收到supervisord的signal

centos6.5 启动docker出现错误symbol dm_task_get_info_with_deferred_remove, version Base not defined in file libdevmapper.so.1.02 with link time reference

yum update device-mapper

6.5的内核必须2.6.32-431或以上才能安装


手动安装docker
 访问https://get.docker.com/builds/,然后根据描述进行安装

docker-machine

  安装
  curl -L https://github.com/docker/machine/releases/download/v0.8.1/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine && \
    chmod +x /usr/local/bin/docker-machine

  docker-machine create --driver virtualbox default
  
  docker-machine -s "/dat/zw-test/.docker" create -d virtualbox --engine-opt bip=10.33.100.1/16 --virtualbox-cpu-count 12 --virtualbox-disk-size 100000 --virtualbox-memory 32768 default
  
  docker-machine -s "/dat/zw/.docker" create -d virtualbox --engine-opt bip=10.33.100.1/16 --virtualbox-cpu-count 30 --virtualbox-disk-size 70000 --virtualbox-memory 40960 default
  
    创建时如果使用-s指定存储路径, 后续所有的命令都需要增加-s;
    
  mount -t vboxsf -o defaults,uid=`id -u docker`,gid=`id -g docker` dat /dat
  
  使用virtualbox的shared folder
    首先在Boot2Docker中mount sharedfolder
    docker-machine ssh
    mkdir /dat
    mount -t vboxsf -o defaults,uid=`id -u docker`,gid=`id -g docker` dat /dat
    以上的方式在重启之后将消失, 使用以下脚本可自动mount
    /mnt/sda1/var/lib/boot2docker/bootlocal.sh
      mkdir /dat
      mount -t vboxsf -o defaults,uid=`id -u docker`,gid=`id -g docker` dat /dat

docker手动指定ip
  首先创建子网
    docker network create --subnet=172.18.0.0/16 mynet123
  使用
    docker run --net mynet123 --ip 172.18.0.22 -it ubuntu bash

Dockerfile
  ADD指令会自动解压gzip, bzip2 or xz文件, 如果不希望自动解压,可以使用COPY
  
清除none:none images
docker rmi $(docker images -f "dangling=true" -q)


lvs
  yum -y install ipvsadm
  
  开启端口转发
    编辑/etc/sysctl.conf文件
      net.ipv4.ip_forward = 1
    sysctl -p
    
    service ipvsadm start
    chkconfig ipvsadm on
    
    清除映射
      ipvsadm -C
    
    添加虚拟服务
      ipvsadm -A -t 172.17.230.255:1935 -s wlc 
    添加实际服务器
      # [ipvsadm -a -t (Service IP:Port) -r (Real Server's IP:Port) -m] ("m" means masquerading (NAT))
      ipvsadm -a -t 172.17.230.255:1935 -r 192.168.99.100:1955 -m
      ipvsadm -a -t 172.17.230.255:1935 -r 192.168.99.100:1965 -m
    查看配置
      ipvsadm -ln
    保存配置
      /etc/rc.d/init.d/ipvsadm save 或 service ipvsadm save
      

      iptables -t nat -A PREROUTING -p tcp --dport 1935 -j DNAT --to-destination 192.168.99.100:1955
      
      iptables -A FORWARD -m state -p tcp -d 192.168.99.100:1955 --dport 1935 --state NEW,ESTABLISHED,RELATED -j ACCEPT
    
          
iptables -L -t nat
iptables -t nat

iptables -t nat -L --line-numbers

iptables -L --line -n -v

iptables -L -v

清除统计数据
  iptables -Z

删除nat rule
  iptables -t nat -D PREROUTING 1

拒绝指定ip连接到端口
  iptables -A INPUT -p tcp --dport 1935 -s 172.28.100.255 -j DROP
阻止连接到指定的端口
  iptables -A INPUT -p tcp --dport 1935 -j DROP


VBoxManage modifyvm "default" --natpf1 "f1,tcp,,1955,,1955"
VBoxManage modifyvm "default" --natpf1 "f2,tcp,,1956,,1956"




centos7 docker
  安装
    curl -sSL https://get.docker.com/ | sh
  启动
    systemctl start docker
    systemctl enable docker
  配置ip及存储
    docker默认的存储在/var/lib/docker, 编辑/var/lib/systemd/system/docker.service
    在dockerd后增加参数
    ExecStart=/usr/bin/dockerd --bip=10.44.0.1/16 --graph=/home/zw/.docker
    
    
    
