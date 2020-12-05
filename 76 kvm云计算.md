## 1.什么是云计算？

云计算是一种按量付费的模式！云计算的底层是通过虚拟化技术来实现的！

####  云计算的服务类型

IAAS 基础设施即服务(infrastructure as an service) 虚拟机 ecs openstack

PAAS 平台即服务(platform as an service ) php，java docker容器

SAAS 软件即服务 企业邮箱服务 cdn服务 rds数据库 开发+运维2.1 

#### 为什么要使用云计算

1. 为了使服务器得到充分的利用，隔离出干净的虚拟环境，让服务于服务之间不互相干扰。

   小公司：前期投入小，扩展灵活，风险小

   大公司：闲置服务器计算资源，虚拟机，出租（超卖）

   公有云：可以租

   私有云：共公司内部使用

   混合云：有自己的私有云+租的公有云

#### 4.云计算的基础KVM虚拟化

***4.1 什么是虚拟化？***

虚拟化，通过模拟计算机的硬件，来实现在同一台计算机上同时运行多个不同的操作系统的技术。

***4.2 虚拟化软件的差别***

linux虚拟化软件：

qemu 软件纯模拟全虚拟化软件，特别慢！AIX，兼容性好！ 

xen(半) 性能特别好，需要使用专门修改之后的内核，兼容性差！redhat 5.5 xen kvm 

KVM（linux） 全虚拟机，它有硬件支持cpu，基于内核，而且不需要使用专门的内核 centos6 kvm 性能较好，兼容较好

vmware workstations: 图形界面

virtual box: 图形界面 Oracle

***4.3 安装kvm虚拟化管理工具***

KVM：全称Kernel-based Virtual Machine

- 基础环境安装

```bash
yum install libvirt virt-install qemu-kvm -y
```

libvirt 作用：虚拟机的管理软件 libvirt: kvm,xen,qemu,lxc....

virt virt-install virt-clone 作用：虚拟机的安装工具和克隆工具 

qemu-kvm qemu-img (qcow2,raw)作用：管理虚拟机的虚拟磁盘

![1574863702832](F:\typora\1574863702832.png)

centos 7.4 7.6

vmware 宿主机 kvm虚拟机

内存4G，cpu开启虚拟化

![1574863749478](F:\typora\1574863749478.png)

***4.4 安装一台kvm虚拟机***

​	分发软件TightVNC或者VNC-Viewer-6.19.325 宿主机
​	微软的远程桌面
​	vnc:远程的桌面管理工具 向日葵 微软的远程桌面

```bash
systemctl start libvirtd.service 
systemctl status libvirtd.service
```

10.0.0.100 宿主机
建议虚拟机内存不要低于1024M，否则安装系统特别慢！

```bash
#创建虚拟机，需要下载最下的linux镜像
virt-install --virt-type kvm --os-type=linux --os-variant rhel7 --name centos7 --memory 1024 --vcpus 1 --disk /opt/centos2.raw,format=raw,size=10 --cdrom /opt/CentOS-7-x86_64-Minimal-1511.iso --network network=default --graphics vnc,listen=0.0.0.0 --noautoconsole

--virt-type kvm 		虚拟化的类型(qemu) 
--os-type=linux 		系统类型 
--os-variant rhel7 		系统版本 
--name centos7 			虚拟机的名字 
--memory 1024 			虚拟机的内存 
--vcpus 1 				虚拟cpu的核数 
--disk /opt/centos2.raw,format=raw,size=10 
--cdrom /opt/CentOS-7-x86_64-DVD-1708.iso 
--network network=default 	使用默认NAT的网络 
--graphics vnc,listen=0.0.0.0 --noautoconsole
```

vnc:10.0.0.100:5900

![1574864331877](F:\typora\1574864331877.png)

***4.5 kvm虚拟机的virsh日常管理和配置***

- 日常管理命令

```bash
virsh list					#查看开机或挂起状态的虚拟机
virsh list --all 			#查看所以虚拟机
virsh shutdown centos7		#关机
virsh destroy centos7 		#快速关机(声音慎用)
virsh reboot centos7		#重启(当安装系统时退出，将不会再弹出安装界面，需要删除重新安装 virsh destroy centos7 强行退出，还需删除磁盘)
virsh undefine centos7_1 	#删除虚拟机
virsh dumpxml web01			#从磁盘文件中导出配置文件
virsh dumpxml web01 >vm_web01.xml

#配置文件和磁盘文件从在可以将删除的虚拟机恢复
virsh define vm_web01.xml

#当虚拟机配置文件被删除时，可以通过磁盘文件恢复
virt-install --virt-type kvm --os-type=linux --os-variant rhel7 --name centos7 --memory 1024 --vcpus 1 --disk /opt/centos2.raw,format=raw,size=10 --boot hd --network network=default --graphics vnc,listen=0.0.0.0 --noautoconsole

-------------------------------------------------
virsh domrename centos7 web01		#主机重命名(需要关机)
`当磁盘文件修改名称后需要修改对应的配置文件内的磁盘名称`
 virsh edit web01		#进入编辑模式，编辑磁盘的配置文件，具有语法检测功能
 匹配/disk找到 <source file='/opt/web01.raw'/>
```

- 日常管理命令2

```bash
virsh suspend web01			#挂起虚拟机，挂机后将不能在vnc操作并且时间停留在挂起的那一刻
virsh resume web01			#恢复挂起的虚拟机

yum install ntpdate -y      #安装ntp服务
ntpdate ntp6.aliyun.com 	#进行时间同步，可以写定时任务

virsh vncdisplay web01 		#查看vnc端口号
```

- 虚拟机开机自启

```bash
virsh autostart wen01 		#设置开机自启（前提是libvirt开机自启）
virsh autostart --disable web01 #取消开机自启

#开机自启原理
[root@kvm01 opt]# ln -s /etc/libvirt/qemu/web01.xml /etc/libvirt/qemu/autostart/
`可以通过查看/etc/libvirt/qemu/autostart/下面的软链接看那些虚拟机开机自启`

```

- console控制台登录

```bash
grubby --update-kernel=ALL --args="console=ttyS0,115200n8"
reboot
```

PS:将VMware挂起后会改变内核转换参数

```bash
[root@kvm01 opt]# sysctl net.ipv4.ip_forward = 1
```

***4.6 kvm虚拟机虚拟磁盘格式转换和快照管理***

- raw： 

  裸格式，占用空间比较大，不支持快照功能，不方便传输 ,读写性能较好 总50G 占用50G,传输50G 

- qcow2： 

  qcow（copy on write）占用空间小，支持快照，性能比raw差一点，方便传输 总50G 占用2G,传输2G

4.6.1磁盘工具的常用命令

```bash
qemu -img info,create，resize，convert

#查看虚拟磁盘信息 
qemu-img info test.qcow2

#创建一块qcow2格式的虚拟硬盘： 
qemu-img create -f qcow2 test.qcow2 2G

#调整磁盘磁盘容量 
qemu-img resize test.qcow2 +20G

#raw转qcow2：
qemu-img convert -f raw -O qcow2 oldboy.raw oldboy.qcow2 
convert [-f fmt] [-O output_fmt] filename 
 
virsh edit web01:
<disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/data/centos2.qcow2'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
</disk>
#关机重启
virsh destroy web01 virsh start web01


output_filename -c 压缩
```

**4.6.2 快照管理**

```bash
#创建快照
virsh snapshot-create-as centos7 --name install_ok 
#查看快照
virsh snapshot-list centos7

#还原快照
virsh snapshot-revert centos7 --snapshotname 1516574134 #删除快照
virsh snapshot-delete centos7 --snapshotname 1516636570

raw不支持做快照，qcow2支持快照，并且快照就保存在qcow2的磁盘文件中
```

- 当删除虚拟机后只有磁盘文件怎样恢复

```bash
virt-install --virt-type kvm --os-type=linux --os-variant rhel7 --name web04 --memory 1024 --vcpus 1 --disk /opt/web04.qcow2 --boot hd --network network=default --graphics vnc,listen=0.0.0.0 --noautoconsole
```

***4.7 kvm虚拟机克隆***

*4.7.1完整克隆*

```bash
#自动创建
virt-clone --auto-clone -o web01 -n web02 (完整克隆)

#手动创建
cp web01.qcow2 web02.qcow2
virsh dumpxml web01 >web02.xml
vim web02.xml
#修改虚拟机的名字
#删除虚拟机uuid
#删除mac地址
#修改磁盘路径
virsh define web02.xml 
virsh start web02
```

*4.7.2 链接克隆*

```bash
#生成虚拟机磁盘文件 
qemu-img create -f qcow2 -b 49-web03.qcow2 49-web04.qcow2

```



```bash
#生成虚拟机的配置文件
virsh dumpxml web01 >web03.xml
vim web03.xml
#修改虚拟机的名字
<name>web03</name>
#删除虚拟机uuid
<uuid>8e505e25-5175-46ab-a9f6-feaa096daaa4</uuid>
#删除mac地址
<mac address='52:54:00:4e:5b:89'/>
#修改磁盘路径
<source file='/opt/web03.qcow2'/>
```

全自动链接克隆脚本：

```bash
[root@kvm01 scripts]# cat link_clone.sh 
#!/bin/bash
old_vm=$1
new_vm=$2
#a：生成虚拟机磁盘文件
old_disk=`virsh dumpxml $old_vm|grep "<source file"|awk -F"'" '{print $2}'`
disk_tmp=`dirname $old_disk`
qemu-img create -f qcow2 -b $old_disk  ${disk_tmp}/${new_vm}.qcow2
#b：生成虚拟机的配置文件
virsh dumpxml $old_vm >/tmp/${new_vm}.xml
#修改虚拟机的名字
sed -ri "s#(<name>)(.*)(</name>)#\1${new_vm}\3#g" /tmp/${new_vm}.xml
#删除虚拟机uuid
sed -i '/<uuid>/d' /tmp/${new_vm}.xml
#删除mac地址
sed -i '/<mac address/d' /tmp/${new_vm}.xml
#修改磁盘路径
sed -ri "s#(<source file=')(.*)('/>)#\1${disk_tmp}/${new_vm}.qcow2\3#g" /tmp/${new_vm}.xml
#c：导入虚拟机并进行启动测试
virsh define /tmp/${new_vm}.xml
virsh start ${new_vm}
```

自己写的

```bash
[root@kvm01 srv]# cat scripts/link_clone.sh 
#!/bin/bash
if_() {
	if [ -z $yuan_host ];then
		echo "请不要输入空值"
		exit 1
	fi
}
model_1() {
	read -p "请输入你要克隆的源主机：" yuan_host
	if_
	read -p "请输入你要克隆的主机名：" link_host
	if_
}

model_2() {
path=/opt
if [ -f ${path}/${yuan_host}.qcow2 ];then
	if [ -s ${path}/${yuan_host}.qcow2 ];then
		qemu-img create -f qcow2 -b ${path}/${yuan_host}.qcow2 ${path}/${link_host}.qcow2
		virsh dumpxml ${yuan_host} >${path}/${link_host}.xml 
		sed  -i "/<name>/c <name>${link_host}</name>" ${path}/${link_host}.xml
		sed -i '/<uuid>/d' ${path}/${link_host}.xml 
		sed -i '/<mac/d' ${path}/${link_host}.xml
		sed -i "/<source file/c <source file='/opt/${link_host}.qcow2'/>" ${path}/${link_host}.xml
		virsh define ${path}/${link_host}.xml 
		virsh start ${link_host}
	else
		echo "磁盘文件为空"
		exit 2
	fi
else
	echo "源主机不存在"
	exit 3	
fi
}
model_1
model_2

```



















```bash
 
 
   76  virsh destroy centos7_1 
   77  virsh list --all \
   78  virsh list --all 
   79  virsh start centos7_1 
   80  virsh shutdown centos7_1 
   81  virsh destroy centos7_1 
   82  
   83  virsh list --all
   84  ls
   85  history 

  102  virsh list --all
  103  virt-install --virt-type kvm --os-type=linux --os-variant rhel7 --name centos7 --memory 1024 --vcpus 1 --disk /opt/centos2.raw,format=raw,size=10 --boot --network network=default --graphics vnc,listen=0.0.0.0 --noautoconsole
  104  virt-install --virt-type kvm --os-type=linux --os-variant rhel7 --name centos7 --memory 1024 --vcpus 1 --disk /opt/centos2.raw,format=raw,size=10 --boot hd --network network=default --graphics vnc,listen=0.0.0.0 --noautoconsole
  105  virsh list --all
  106  ls
  107  #virsh domrename centos7 web01
  108  virsh destroy centos7 
  109  
  110  virsh list --all 
  111  mv centos2.raw web01.raw
  112  virsh start web01 
  113  virsh edit web01
  114  virsh start web01 
  115  virsh list -all
  116  virsh list --all
  117  history 

```

```bash


qemu-img create -f qcow2 -b web02.qcow2 web04.qcow2
sed  -n '/<name>/c <name>web03</name>' web03.xml
sed -i '/<uuid>/d' web03.xml 
sed -i '/<mac/d' web03.xml
sed -i '/<source file/c <source file='/opt/web03.qcow2'/>' web03.xml
awk -F "['.]" '/<source file/ {print $2}' web03.xml
```








