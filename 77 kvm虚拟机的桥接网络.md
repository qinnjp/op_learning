#### 4.8 kvm虚拟机的桥接网络

- 虚拟机桥接网络配置

```bash
[root@nfs ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0 
TYPE=Ethernet
PROXY_METHOD=dhcp
NAME=eth0
ONBOOT=yes
```

![1574939887801](F:\typora\1574939887801.png)

虚拟机使用物理网段就是桥接

***4.8.1 kvm虚拟机使用桥接模式***

- 开启桥接网卡

```bash
`使用桥接网卡要保证ip地址为静态ip，而不是DHCP自动获取的`
virsh iface-bridge eth0 br0
#关闭桥接网卡
virsh iface-unbridge br0
```

***4.8.2 新虚拟机设为桥接模式***

- 默认net模式

```bash
virt-install --virt-type kvm --os-type=linux --os-variant rhel7 --name web04 --memory 1024 --vcpus 1 --disk /opt/web04.qcow2 --boot hd --network network=default --graphics vnc,listen=0.0.0.0 --noautoconsole
```

- 桥接模式

```bash
virt-install --virt-type kvm --os-type=linux --os-variant rhel7 --name web04 --memory 1024 --vcpus 1 --disk /data/web04.qcow2 --boot hd --network bridge=br0 --graphics vnc,listen=0.0.0.0 --noautoconsole
#bridge=br0

#配置ip地址
echo 'TYPE="Ethernet"
BOOTPROTO="none"
NAME="eth0"
DEVICE="eth0"
ONBOOT="yes"
IPADDR="10.0.0.101"
NETMASK="255.255.255.0"
GATEWAY="10.0.0.254"
DNS1="223.5.5.5"'  >/etc/sysconfig/network-scripts/ifcfg-eth0
```

***4.8.3 将已有虚拟机网络修改为桥接模式***

- 关机状态下修改虚拟机配置文件：

```
[root@kvm01 opt]# virsh edit web0
<interface type='bridge'>
<source bridge='br0'/>
```

- 启动虚拟机，测试虚拟机网络

`如果上层没有开启dhcp,需要手动配置ip地址`

```bash
echo 'TYPE="Ethernet"
BOOTPROTO="none"
NAME="eth0"
DEVICE="eth0"
ONBOOT="yes"
IPADDR="10.0.0.101"
NETMASK="255.255.255.0"
GATEWAY="10.0.0.254"
DNS1="223.5.5.5"'  >/etc/sysconfig/network-scripts/ifcfg-eth0
```

**PSkvm的网络原理**

![1574941531235](F:\typora\1574941531235.png)



#### 4.9热添加技术

热添加硬盘、网卡、内存、cpu

***4.9.1添加硬盘***

- 创建硬盘

```bash
qemu-img create -f qcow2 /opt/web02_add01.qcow2 10G
```

- 添加硬盘

```bash
#临时添加
virsh attach-disk web02 /opt/web02_add01.qcow2 vdb --sourcetype qcow2
#永久添加
virsh attach-disk web02 /opt/web02_add01.qcow2 vdb --sourcetype qcow2 --config

#临时剥离硬盘
virsh detach-disk web01 vdb
#永久剥离硬盘
virsh detach-disk web01 vdb --config

#格式化硬盘
[root@web02 ~]# mkfs.xfs /dev/vdb
[root@web02 ~]# mount /dev/vdb /mnt

`扩容硬盘`
#剥离硬盘
[root@web02 ~]# virsh detach-disk web01 vdb
#调整容量
[root@web02 ~]# qemu-img resize /opt/web02_add01.qcow2 30G
[root@web02 ~]# qemu-img info /opt/web02_add01.qcow2
#添加硬盘
virsh attach-disk web02 /opt/web02_add01.qcow2 vdb --sourcetype qcow2

#更新分区表信息
[root@web02 ~]# xfs_growfs /dev/vdb
`扩容根分区`(百度)
```

***4.9.2添加网卡***

```bash
#添加
[root@kvm01 opt]# virsh attach-interface web02 --type bridge --source br0

#优化
[root@kvm01 opt]# virsh attach-interface web02 --type bridge --source br0 --model virtio

#永久添加（下一次启动生效）
[root@kvm01 opt]# virsh attach-interface web02 --type bridge --source br0 --model virtio --cinfig

#删除网卡
virsh detach-interface web04 --type bridge --mac 52:54:00:35:d3:71
```

***4.9.3 kvm虚拟机在线热添加内存***

```bash
#临时热添加内存 (只能下调)
virsh setmem web04 1024M 

#永久增大内存 
virsh setmem web04 1024M --config
```

- 直接修改配置文件

```bash
[root@kvm01 opt]# virsh edit web02 
  <memory unit='KiB'>1048576</memory>
  <currentMemory unit='KiB'>1048576</currentMemory>

关机重启
virsh destroy web02
virsh start web02

#调整虚拟机内存最大值
virsh setmaxmem web04 4G
```

***4.9.4 kvm虚拟机在线热添加cpu***

```bash
热添加cpu核数
virsh setvcpus web04 4 
永久添加cpu核数 
virsh setvcpus web04 4 --config

#调整虚拟机内核数
[root@kvm01 opt]# virsh setvcpus --maximum web02 4 --config
重启生效


```

#### 4.10 virt-manager和kvm虚拟机热迁移(共享的网络文件系统)

`准备`

| 主机名 | ip        | 内存 | 网络            | 软件需求 | 虚拟化     |
| :----- | :-------- | :--- | :-------------- | :------- | :--------- |
| kvm01  | 10.0.0.11 | 2G   | 创建br0桥接网卡 | kvm和nfs | 开启虚拟化 |
| kvm02  | 10.0.0.12 | 2G   | 创建br0桥接网卡 | kvm和nfs | 开启虚拟化 |
| nfs01  | 10.0.0.31 | 1G   | 无              | nfs      | 无         |

```bash
yum install libvirt virt-install qemu-kvm -y
```



***1 冷迁移kvm虚拟机:配置文件,磁盘文件***

```bash
scp -rp /opt/web02.qcow2 10.0.0.99:/opt
virsh dumpxml web02>web02.xml
scp web02.xml 10.0.0.99:/opt
virsh define web02.xml
```

***2.热迁移kvm虚拟机:配置文件,nfs共享***

 ```bash
1.配置nfs服务器
[root@nfs ~]# cat /etc/exports
/vm 10.0.0.0/24(rw,async,no_all_squash,no_root_squash)
[root@nfs ~]# mkdir /vm
[root@nfs ~]# systemctl restart nfs

2.移走/opt/的文件，挂载nfs
[root@kvm01 opt]# pkill qemu-kvm
[root@kvm01 opt]# rm -rf /etc/libvirt/qemu/*.xml
[root@kvm01 opt]# systemctl restart libvirtd
[root@kvm01 opt]# mv /opt/* /srv/
[root@kvm01 opt]# mount -t nfs 10.0.0.31:/vm /opt
[root@kvm01 opt]# mv /srv/web01.qcow2 /srv/web01.xml /opt/
[root@kvm01 opt]# virsh define web01.xml 
[root@kvm01 opt]# virsh start web01
 ```

- 在线热迁移

```bash
wget http://192.168.37.202/linux59/web04.qcow2 virt-install --virt-type kvm --os-type=linux --os-variant rhel7 --name web04 --memory 1024,maxmemory=2048 --vcpus 1,maxvcpus=10 --disk /data/web04.qcow2 --boot hd --network bridge=br0 --graphics vnc,listen=0.0.0.0 --noautoconsole

#临时迁移
virsh migrate --live --verbose web04 qemu+ssh://10.0.0.11/system --unsafe

#永久迁移
virsh migrate --live --verbose web03 qemu+ssh://10.0.0.100/system --unsafe --persistent --undefinesource
```

















