---
layout: "post"
title: "virtualbox"
date: "2016-06-22 16:45"
---

查看vm

	vboxmanage list vms
	vboxmanage list runningvms
	
查看指定的虚拟机配置
	vboxmanage showvminfo centos6.6-2
	
添加端口转发
	vboxmanage modifyvm --natpf<1-N> [<name>],tcp|udp,[<hostip>],<hostport>,[<guestip>], <guestport>
	VBoxManage modifyvm "VM name" --natpf1 "guestssh,tcp,,2222,,22"
	VBoxManage modifyvm "VM name" --natpf1 "guestssh,tcp,,2222,10.0.2.19,22"

添加bridge网络
	vboxmanage modifyvm default --nic3 bridged --bridgeadapter3 eth0
	
删除网络接口
	vboxmanage modifyvm "VM name" --nic3 none
	
删除端口转发
	VBoxManage modifyvm "VM name" --natpf1 delete "guestssh"
	
删除网卡
	VBoxManage hostonlyif remove vboxnet4
	
导入
	查看信息
		vboxmanage import xxx.ovf -n
	导入
		vboxmanage import xxx.ovf
		
导出
	vboxmanage export win7-32 -o win-32.83.ovf
	
属性
	设置默认vm目录
		vboxmanage setproperty machinefolder /path/xx
	获得属性
		vboxmanage list systemproperties
		
修改
	设置cpu个数
		开启热拔插 modifyvm win7-32 --cpuhotplug on
		设置个数 modifyvm "vmname" --cpus 2
		
启动/停止
	
	startvm win7-32 --type headless
	controlvm win7-32 poweroff
	
删除	
	vboxmanage unregistervm --delete "centos6.6-2"
	如果出现被锁定,则需要先执行以下命令
	vboxmanage controlvm "centos6.6-2" poweroff	

添加共享目录
	vboxmanage sharedfolder add default --name dat --hostpath /dat --automount

在虚拟机中挂载shared folder
	mount -t vboxsf dev /Volumes/DATA/dev
	
