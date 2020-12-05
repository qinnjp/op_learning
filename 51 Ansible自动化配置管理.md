## Ansible自动化配置管理

安装 配置 启动

### 课程大纲

***1.什么是ansible？***

Ansible是一个IT自动化管理工具，可以通过一个命令完成一系列的操作。进而减少我们重复性的工作和维护成本，以提高工作的效率。

***2.ansible优点特点？***

- 优点

1. 批量执行远程命令
2. 批量配置软件服务
3. 实现软件开发功能
4. 编排高级的IT任务

- 特点

1. 容易学习无代理模式
2. 操作灵活，体现在有较多的模块提供了丰富的功能
3. 简单易用
4. 安全可靠
5. 移植性高

***3.ansible基础架构？控制端 被控端 inventory ad-hoc playbook 连接协议？***

![1570619397006](F:\typora\1570619397006.png)

***4.ansible配置文件？***

```bash
#优先级顺序从上到下
ANSIBLE_CONFIG				#变量
ansible.cfg 				#当前项目目录中
.ansible.cfg 				#当前执行用户的家目录
/etc/ansible/ansible.cfg	#默认配置文件

#测试变量优先级  
[root@manager ~]# export ANSIBLE_CONFIG="/tmp/ansible.cfg"
[root@manager ~]# touch /tmp/ansible.cfg
[root@manager ~]# ansible --version

#测试当前目录的
[root@manager ~]# mkdir /project1
[root@manager ~]# cd /project1/
[root@manager project1]# touch ansible.cfg
[root@manager project2]# ansible --version
ansible 2.8.5
config file = /project1/ansible.cfg
[root@manager /]# mkdir /project2
[root@manager /]# cd /project2/
[root@manager project2]# touch ansible.cfg
[root@manager project1]# ansible --version
ansible 2.8.5
config file = /project2/ansible.cfg

#测试当前用户家目录
[root@manager tmp]# touch ~/.ansible.cfg
[root@manager tmp]# ansible --version
ansible 2.8.5
config file = /root/.ansible.cfg
```

PS：通常我们做项目时，通常会将ansible.cfg	hosts	playbook 放在一个目录下面。便于后期的打包推送。

***5.ansible inventory主机清单***

```bash
#1.基于IP地址+密码的方式
[webservers]
172.16.1.7 ansible_ssh_user='root' ansible_ssh_pass='1'
172.16.1.8 ansible_ssh_user='root' ansible_ssh_pass='1'

#第一次ssh连接时不需要认证
host_key_checking = False

#测试
[root@manager project1]# ansible webservers -m ping -i hosts

#inventory      = /etc/ansible/hosts   #定义hosts路径不需要-i

2.场景二、基于密钥连接，需要先创建公钥和私钥，并下发公钥至被
控端
[root@manager ~]# ssh-keygen -C manager@qq.com	#配置公钥私钥 一路回车
[root@manager ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.16.1.7
[root@manager ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.16.1.8
#方式一、主机+端口+密钥
[root@manager ~]# cat hosts
[webservers]
172.16.1.7
172.16.1.8

`ansible批量推送ssh公钥`
#########################
## hosts
[webserver]
172.16.1.7   ansible_user=root  ansible_ssh_pass="1"
172.16.1.8   ansible_user=root  ansible_ssh_pass="1"
cat ssh.ymal
- hosts: all
  tasks:
    - name: Deliver Authorized_key
      authorized_key:
        user: root
        key: "{{ lookup ( 'file', '/root/.ssh/id_rsa.pub' ) }}"
        state: present
        exclusive: yes


3.场景三、主机组使用方式
[lbservers] #定义lbservers组
172.16.1.5
172.16.1.6

[webservers] #定义webserver组
172.16.1.7
172.16.1.8

[servers:children] #定义servers组包括两个子组
[lbservers,webserver]
lbservers
webserver

#查看主机组中有哪些主机
[root@manager project1]# ansible webservers --list-hosts -i hosts
hosts (2):
172.16.1.7
172.16.1.8
[root@manager project1]# ansible all --list-hosts -i hosts #查看所有组的主机

```

***6.ansible ad-hoc？ 单条命令***

1. 什么是ad-hoc

   ad-hoc简而言之就是“临时命令”，执行完即结束，并不会保存

2. ad-hoc模式的使用场景

   比如查看多台机子上的某个进程是否启动，或拷贝指定文件到本地

3. ad-hoc模式的命令使用，ansible qjp -m command -a "df -h",含义如下图

![1570622090771](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1570622090771.png)



4. 使用ad-hoc执行一次远程命令，注意观察返回结果的颜色

   绿色：代表被管理主机没有被修改

   黄色：代表被管理主机发现变更

   红色：代表出现了故障，注意查看提示

5. 学习的模块

```bash
command 				#执行命令 默认 不支持管道
shell 					#执行命令 支持管道
yum_reposity 			#yum仓库配置
yum						#yum安装软件
get_url 				#和linux的wget一致
copy 					#拷贝配置文件
service|systemd 		#启动服务
user					#创建用户
group					#创建组
file 					#创建目录 创建文件 递归授权
mount 					#挂载
cron 					#定时任务
firewalld 				#防火墙
selinux 				#selinuix
```

1.command

```bash
[root@manager project1]# ansible webservers -a "ls" -i hosts	#默认使用模块command，不支持管道
```

2.shell

```bash
[root@manager project1]# ansible webservers -m shell -a "ps aux|grep nginx" -i hosts	#支持管道
```

3.yum

```bash
state:
	present 			安装
	absent 				卸载
	latest 				最新
	enablerepo 			#指定使用按个仓库
	disablerepo 		#排除使用哪个仓库

#1.安装最新的httpd服务
[root@manager project1]# ansible webservers -m yum -a "name=httpd state=latest disablerepo=webtaticphp" -i hosts

#2.移除httpd服务
[root@manager project1]# ansible webservers -m yum -a "name=httpd state=absent disablerepo=webtaticphp" -i hosts

#3.安装httpd指定从按个仓库安装
- name: install the latest version of Apache from
the testing repo
[root@manager project1]# ansible webservers -m yum -a "name=httpd state=latest enablerepo=testing" -i
hosts

#4.通过URL方式进行安装
[root@manager project1]# ansible webservers -m yum -a "name=https://mirrors.aliyun.com/zabbix/zabbix/3.0/ rhel/7/x86_64/zabbix-agent-3.0.0-1.el7.x86_64.rpm state=present disablerepo=webtatic-php" -i hosts
- name: install nginx rpm from a local file (软件包
必须在被控端主机)
[root@manager project1]# ansible webservers -m yum -a "name=/root/zabbix-agent-4.0.0-2.el7.x86_64.rpm state=present disablerepo=webtatic-php" -i hosts

```

4.copy

```bash
src 			#本地路径,可以是相对,可以是绝对
dest 			#目标位置
owner		 	#属主
group 			#属组
mode 			#权限
backup 			#备份

[root@manager project1]# ansible webservers -m copy -a "src=./file/ansible.oldxu.com.conf dest=/etc/nginx/conf.d/ansible.oldxu.com.conf owner=root group=root mode=644" -i hosts

[root@manager project1]# ansible webservers -m copy -a "src=./file/ansible.oldxu.com.conf dest=/etc/nginx/conf.d/ansible.oldxu.com.conf owner=root group=root mode=644 backup=yes" -i hosts
```



5.systemd

```bash
state
	started 		#启动
	stopped 		#停止
	restarted 		#重启
	reloaded 		#重载
enabled 			#是否开机自启
	yes 			#是
	no 				#否

[root@manager project1]# ansible webservers -m systemd -a "name=nginx state=restarted enabled=yes" -i hosts

```

6.file

```bash
#创建 /code/ansible

path #路径
state
	touch 					#创建文件
	directory 				#创建目录
owner 						#属主
group 						#属组
mode 						#权限

#准备站点
[root@manager project1]# ansible webservers -m file -a "path=/code/ansible state=directory mode=755 owner=www group=www" -i hosts

#准备站点代码
[root@manager project1]# ansible webservers -m copy -a "src=./file/index.html dest=/code/ansible/index.html owner=www group=www mode=644" -i hosts

```

7.user group

```bash
#group   整数int   小数   flot   dasdsa   str   真|假  bool

[root@manager project1]# ansible webservers -m
group -a "name=www gid=666 state=present" -i hosts

#user
name 			#名称
uid 			#uid
group 			#组名或gid
create_home 	#是否创建家目录
system 			#是否作为系统组
shell 			#指定登录shell
state
	present
	absent
remove
groups
append
password
#--------------------------------------------------------------->

# 程序使用 www 666 666 /sbin/nologin /home  -->无

[root@manager project1]# ansible webservers -m user -a "name=www uid=666 group=666 create_home=no shell=/sbin/nologin state=present" -i hosts

# 正常用户 oldxu 1000 1000 /bin/bash /home/oldxu
[root@manager project1]# ansible webservers -m user -a "name=oldxu" -i hosts

# 移除oldxu用户,并删除家目录所有内容.
[root@manager project1]# ansible webservers -m user -a "name=oldxu state=absent remove=yes" -i hosts

# 创建 other用户.有两个附加组root bin,创建家目录,指定登录shell,设定密码123
#生成一个密码
ansible all -i localhost, -m debug -a "msg={{ '123' | password_hash('sha512', 'mysecretsalt') }}"

[root@manager project1]# ansible webservers -m user -a 'name=other groups='root,bin' create_home=yes shell=/bin/bash password="$6$mysecretsalt$gIIYs0Xgc7sSQkH.zKaz8/Afa MomYzR1QZYtccwmJcUt8VpLq4D055UCCX4MlwgePOP80ZRwhppv BF72RIAVi/"' -i hosts

```

8.mount

```bash
#提前准备好nfs服务端
[root@web01 ~]# showmount -e 172.16.1.31
Export list for 172.16.1.31:
/data/zrlog 172.16.1.0/24

#用管理端操作被控端,让被控端挂载nfs存储数据
present #写入/etc/fstab
absent #卸载/etc/fstab

mounted #临时挂载
unmounted #卸载当前挂载

#挂载过程中,如果目录不存在,则会创建该目录
[root@manager project1]# ansible webservers -m mount -a "src=172.16.1.31:/data/zrlog path=/test_zrlog fstype=nfs opts=defaults state=mounted" -i hosts

[root@manager project1]# ansible webservers -m mount -a "src=172.16.1.31:/data/zrlog path=/test_zrlog fstype=nfs opts=defaults state=unmounted" -i hosts

```

9.cron

```bash
minute 		#分
hour 		#时
day 		#日
month 		#月
week 		#周
job 		#

[root@manager project1]# ansible webservers -m cron -a 'name=test_job minute=00 hour=02 job="/bin/bash /server/scripts/client_to_data_server.sh &>/dev/null"' -i hosts

[root@manager project1]# ansible webservers -m cron -a 'name=test job="/bin/bash /server/scripts/test.sh &>/dev/null"' -i hosts

[root@manager project1]# ansible webservers -m cron -a 'name=test job="/bin/bash /server/scripts/test.sh &>/dev/null" state=absent' -i hosts
```

10.firewalld

```bash
[root@manager project1]# ansible webservers -m systemd -a "name=firewalld state=started" -i hosts

#针对服务
[root@manager project1]# ansible webservers -m firewalld -a "service=http state=enabled" -i hosts

#针对端口
[root@manager project1]# ansible webservers -m firewalld -a "port=9999/tcp state=enabled" -i hosts

#针对source来源
[root@manager project1]# ansible webservers -m firewalld -a 'source="10.0.0.0/24" zone=trusted state=enabled permanent=yes' -i hosts

#针对rule
[root@manager project1]# ansible webservers -m firewalld -a 'rich_rule="rule family=ipv4 source address=10.0.0.1/32 service name=http accept" state=enabled' -i hosts
```

11.selinux

```bash
[root@manager project1]# ansible webservers -m selinux -a "state=disabled" -i hosts
```







1.安装http服务
2.编写简单网页测试内容
3.启动服务并加入开机自启
4.放行对应的端口