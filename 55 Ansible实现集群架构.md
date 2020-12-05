## Ansible实现集群架构

***1.Ansible基础优化***

1. 准备roles目录

~~~bash
[root@manager ~]# mkdir /opt/roles/
~~~

2.准备清单文件

```bash
[root@manager roles]# cat /opt/roles/hosts
[nfsservers]
172.16.1.31

[backupservers]
172.16.1.41

[lbservers]
172.16.1.5
172.16.1.6

[webservers]
172.16.1.7
172.16.1.8

[dbservers]
172.16.1.51
```

3.准备ansible配置文件

```bash
[root@manager roles]# cat /opt/roles/ansible.cfg
[defaults]
inventory      = ./hosts
host_key_checking = False
forks          = 100
```

4.与每台主机建立密匙连接

~~~bash
#创建
[root@manager ~]# ssh-keygen -C manager@qq.com

#推送
[root@manager ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.16.1.7
~~~



5.测试主机连通性

```bash
[root@manager roles]# ansible all --list-hosts
  hosts (7):
    172.16.1.41
    172.16.1.7
    172.16.1.8
    172.16.1.5
    172.16.1.6
    172.16.1.51
    172.16.1.31
```

***2.架构基础环境***
	- 防火墙					firewalld selinux 关闭
	- yum源  				base epel  nginx php
	- 安装软件	
	- 用户创建			www
	- SSH配置
	- 内核参数
	- 文件描述符
	- rsync备份脚本

