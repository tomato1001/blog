---
layout: post
title: vagrant与virtualbox自定义centos7虚拟机
categories: ['vagrant', 'virtualbox']
date: 2015-08-15 10:46:47
---

1. 首先安装virtualbox和vagrant

2. 通过virtualbox创建一个虚拟主机

	虚拟机配置如下：

		* 名称：centos7
		* 类型：Linux
		* 版本：RedHat 64bit
		* 内存：512MB
		* 虚拟硬盘：40G，硬盘类型选择**vmdk**，尺寸动态分配

	修改硬件设置，开启ssh端口转发

		* 关闭audio
		* 关闭usb
		* 将第一个网卡设置为**网络地址转换NAT**并配置端口转发

				* 名称：SSH
				* 协议：TCP
				* 主机端口：2222
				* 子系统端口：22
				* 主机ip：
				* 子系统ip：

		* 将第二个网卡设置为Host-only，便于通过虚拟机访问主机

	安装centos7

		* 安装时创建用户vagrant，密码vagrant
		* root用户密码可以随意设置，也可以设置为vagrant

	visudo设置

		因为vagrant需要在运行sudo时不提示密码，所以需要通过visudo进行设置
				su - root
				visudo

		添加或修改成如下配置：

				Defaults:vagrant !requiretty
				#Defaults !visiblepw
				vagrant ALL=(ALL) NOPASSWD: ALL

		切换到vagrant用户测试修改是否成功
		
				su - vagrant
				sudo pwd

		如果成功打印当前路径，则修改成功，否则检查visudo是否配置正确

	更新系统

			sudo yum update
	更新完成后重启
			sudo reboot
	重启完成之后删除旧的内核(可选操作)
		如果之前更新了内核，可以将旧的内核删除
		查看已安装的内核包
		sudo rpm -q kernel
		sudo rpm -e 需要删除的内核包名称

	为vagrant设置ssh公匙
			su - vagrant
			mkdir -p ~/.ssh
			chmod 0700 ~/.ssh
			wget https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub -O ~/.ssh/authorized_keys
			chmod 0600 ~/.ssh/authorized_keys

	安装OpenSSH server

			sudo yum install openssh-server -y

			使ssh server开机自动启动

					systemctl enable sshd

			配置OpenSSH Server
				sudo vi /etc/ssh/sshd_config
			添加或修改以下行：
				Port 22
				PubKeyAuthentication yes
				AuthorizedKeysFile .ssh/authorized_keys
				PermitEmptyPasswords no
				PasswordAuthentication no
				UseDNS no
			保存文件，重启SSH服务
				sudo systemctl reload sshd

	安装virtualbox guest tools
		
		如果之前执行yum update时更新过内核，必须重启机器并选择从新的内核启动，否则将出现无法找到内核头文件的错误
		增加epel源
			wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm && sudo yum install epel-release-latest-7.noarch.rpm -y
			
		安装依赖
			sudo yum install -y gcc make kernel-devel dkms bzip2
		挂载guest iso
			点击virtualbox的Devices > Insert Guest Additions CD Image
			查看是否已经自动挂载到/mnt或/media目录下，如果没有则执行sudo mount /dev/cdrom /mnt进行手动挂载
			在/mnt下执行sudo ./VBoxLinuxAdditions.run
		关闭虚拟机
			sudo shutdown -h now

	清理资源
		删除/etc/udev/rules.d/70-persistent-net.rules
		删除/etc/sysconfig/network-scripts目录下ifcfg-开头的文件（保留ifcfg-lo和ifcfg-eth0）

	打包
		vagrant package --base centos7(之前创建虚拟机时指定的名称)
	测试
		vagrant box add testbox package.box
		vagrant init testbox
		vagrant up
		vagrant ssh

	问题解决
		当执行vagrant up出现


		The following SSH command responded with a non-zero exit status.
		Vagrant assumes that this means the command failed!

		ARPCHECK=no /sbin/ifup eth1 2> /dev/null

		尝试执行vagrant ssh，然后删除/etc/udev/rules.d/70-persistent-net.rules文件

3. 参数配置

{% highlight ruby %}
config.vm.define "centos6" do |c6|
	c6.vm.network "private_network", ip: "192.168.56.101"
	config.vm.provider "virtualbox" do |vb|
		vb.name = "centos6"
		vb.memory = "1024"
		vb.cpus = 2
	end
end
{% endhighlight %}

	配置root登录
	
		config.ssh.username = 'root'
		config.ssh.password = 'vagrant'
		config.ssh.insert_key = 'true'


4. package相关

需要在对应的vm配置中设置如下配置: config.ssh.insert_key = false然后vagrant up并安装好相关预设程序, 再进行打包操作,否则打包后的vm将无法正常ssh登陆.如果对应的vm之前没有设置config.ssh.insert_key = false,需要将该vm删除,重新建立

5. 完整配置文件

{% highlight sh %}
Vagrant.configure(2) do |config|
	config.vm.box_check_update = false
	config.vm.define "c6" do |c6|
               c6.vm.network "private_network", ip: "10.0.10.100"
               c6.vm.box = "centos6.6"
               c6.ssh.insert_key = false
               c6.vm.provider "virtualbox" do |vb|
                   vb.name = "centos6.6"
                   vb.memory = "2048"
                   vb.cpus = 2
               end
           end
           config.vm.define "c62" do |c62|
           c62.vm.network "private_network", ip: "10.0.10.101"
           c62.vm.box = "centos6.6"
           #c62.ssh.username = "root"
           #c62.ssh.password = "vagrant"
           #c62.ssh.insert_key = true
           c62.vm.network :forwarded_port, guest: 22, host: 2223, host_ip: "0.0.0.0", id: "ssh", auto_correct: true
           c62.vm.provider "virtualbox" do |vb|
               vb.name = "centos6.6-2"
               vb.memory = "2048"
               vb.cpus = 2
           end
         end
end
{% endhighlight %}

6. 自定义vagrant路径
	设置VAGRANT_HOME=/path/to/dir变量