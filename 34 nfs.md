# NFS



## 1.nfs概述

1. 什么是nfs？ 

   network file system 网络文件系统  nfs共享存储

2. nfs能干什么？

   nfs能为不同的主机系统之间实现文件的共享

3. 为什么要使用nfs？

   在集群架构中使用

4. nfs能解决什么问题？

   - 解决多台机器静态资源一致性问题
   - 解决多台机器资源共享
   - 能解决磁盘空间浪费的问题

5. 使用nfs的注意事项？

   - 添加共享存储，只会带来网站的访问延时和消耗，并不会增加网站访问的速度。

   - CDN

     1.购买厂商CDN   --->  用户请求img--->CDN--->负载均衡-->Web-->存储-->CDN缓存该图片

     2.所有的web都是用共享存储,图片此时一致, 只需要将图片定期的推送至CDN

## 2.nfs实现原理

- 本地文件操作方式
  1. 当用户执行mkdir命令，BashShell无法完成该命令操作，会将其翻译给内核。
  2. Kernel内核解析完成后会驱动对应的磁盘设备，完成创建目录的操作。
- NFS实现原理
  1. NFS客户端执行增、删等操作，客户端会使用不同的函数对该操作进行封装。(windows linux mac)
  2. NFS客户端会通过TCP/IP的方式传递给NFS服务端。(可靠)
  3. NFS服务端接收到请求后，会先调用portmap进程进行端口映射。
  4. nfsd进程用于判断NFS客户端是否拥有权限连接NFS服务端。
  5.  Rpc.mount进程判断客户端是否有对应的权限进行验证。读  写
  6.  idmap进程实现用户映射和压缩。
  7.  最后NFS服务端会将客户端的函数转换为本地能执行的命令，然后将命令传递至内核，由内核驱动硬件。

注意: rpc是一个远程过程调用，那么使用nfs必须有rpcbind服务

![1568032229744](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1568032229744.png)

## 3.安装、配置、nfs服务

~~~bash
#1.安装
		[root@nfs ~]# yum install nfs-utils -y
	
#2.配置
	#1.共享什么目录?
	#2.共享给谁使用?
	#3.共享后目录,客户端拥有什么权限?
		[root@nfs ~]# cat /etc/exports
		/data 172.16.1.0/24(rw)
		
#3.根据配置进行初始化环境
		[root@nfs ~]# mkdir /data
		[root@nfs ~]# chown -R nfsnobody.nfsnobody /data/

#4.启动
		[root@nfs ~]# systemctl enable nfs
		[root@nfs ~]# systemctl start nfs	

#5.客户端测试
		[root@backup ~]# yum install nfs-utils -y
		[root@backup ~]# showmount -e 172.16.1.31
		Export list for 172.16.1.31:
		/data 172.16.1.0/24
		
	#挂载远程172.16.1.31的/data至本地的/mnt目录
		[root@backup ~]# mount -t nfs 172.16.1.31:/data /mnt


#6.错误的示范
		#访问被拒绝 (没有允许该网段访问)
	[root@backup ~]# mount -t nfs 10.0.0.31:/data /media/
	mount.nfs: access denied by server while mounting 10.0.0.31:/data


		#能够连接,但是权限被拒绝
	[root@backup mnt]# touch file
	touch: cannot touch ‘file’: Permission denied


#7.多个客户端共享一个存储服务器 (NFS)
#8.实现开机自动挂载(因为服务器不重启)   扩展了解即可
	[root@web01 ~]# cat /etc/fstab
	172.16.1.31:/data			  /media		  nfs     defaults        0 0
~~~

## 4.nfs相关的配置参数

| nfs共享参数    | 参数作用                                                     |
| -------------- | ------------------------------------------------------------ |
| rw*            | 读写权限 (最多)                                              |
| ro             | 只读权限 (只希望看,不希望写)                                 |
| root_squash    | 当NFS客户端以root管理员访问时，映射为NFS服务器的匿名用户nfsnobody(不常用) |
| no_root_squash | 当NFS客户端以root管理员访问时，映射为NFS服务器的root管理员(不常用) |
| no_all_squash  | 无论NFS客户端使用什么账户访问，都不进行用户压缩  ( 后面讲云计算课程会用上 ) |
| all_squash     | 无论NFS客户端使用什么账户访问，均映射为NFS服务器的匿名用户(常用) |
| sync*          | 同时将数据写入到内存与硬盘中，保证不丢失数据                 |
| async          | 优先将数据保存到内存，然后再写入硬盘；这样效率更高，但可能会丢失数据 |
| anonuid*       | 配置all_squash使用,指定NFS的用户UID,必须存在系统             |
| anongid*       | 配置all_squash使用,指定NFS的用户UID,必须存在系统             |

​	

1. rw 和 ro

~~~bash
[root@nfs ~]# cat /etc/exports
/data 172.16.1.0/24(ro)
[root@nfs ~]# systemctl restart nfs

#提示,该目录是一个只读文件
[root@web01 media]# touch file
touch: cannot touch ‘file’: Read-only file system
~~~

2. 验证all_squash  anonuid  anongid

~~~bash
[root@nfs ~]# cat /etc/exports
	/data 172.16.1.0/24(rw,sync,all_squash,anonuid=666,anongid=666)

#1.创建系统真实用户,指定uid和gid为666
[root@nfs ~]# groupadd -g 666 www
[root@nfs ~]# useradd -u666 -g666 www
[root@nfs ~]# id www
uid=666(www) gid=666(www) groups=666(www)

#2.变更属主和属组
[root@nfs ~]# chown -R www.www /data/

#3.重启nfs
[root@nfs ~]# systemctl restart nfs

#4.客户端使用(一定要与服务端使用的匿名用户一致)
[root@web01 ~]# groupadd -g 666 www
[root@web01 ~]# useradd -u666 -g666 www
[root@web01 ~]# mount -t nfs 172.16.1.31:/data/ /media/
	
#5.nfs如何共享多个目录?
[root@nfs ~]# cat /etc/exports
/data 172.16.1.0/24(rw,sync,all_squash,anonuid=666,anongid=666)
/data_2 172.16.1.0/24(rw,sync,all_squash,anonuid=666,anongid=666)
~~~

## 5.nfs的优缺点以及应用环境

- NFS存储优点

1. NFS简单易用、方便部署、数据可靠、服务稳定、满足中小企业需求。

2. NFS的数据都在文件系统之上，所有数据都是能看得见。 

   除了NFS:  		( Glusterfs分布式  赠送 )  MooseFS   FastDFS

- NFS存储局限
  1. 存在单点故障, 本身NFS不支持高可用,也不支持集群.
  2.  NFS数据都是明文，并不对数据做任何校验，也没有密码验证(强烈建议内网使用)。

- NFS应用建议
  1. 生产场景应将静态数据(jpg\png\mp4\avi\css\js)尽可能放置CDN场景进行环境, 以此来减少后端存储压力
  2. 如果没有缓存或架构、代码等，本身历史遗留问题太大，在多存储也没意义

​    NFS就是用来共享  其他什么都没有.     所有的静态都是CDN提供访问的