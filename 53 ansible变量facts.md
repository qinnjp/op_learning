## Ansible varialbes

***1.什么是变量?***

​	以一个固定的字符串,表示一个不固定的值    version:  1.12

***2.定义变量?***

- 1.在playbook中定义变量?

  - vars  关键字

  ```bash
  [root@manager project1]# cat f2.yml 
  - hosts: webservers
    vars:
      - file_name: playbook_vars
  
    tasks:
      - name: Create New File
        file:
          path: /tmp/{{ file_name }}
          state: touch
  ```

  - vars_file   属于一种共享的方式

  ![](E:\老男孩视频\53 老男孩教育-标杆班级-Ansible-variables-day21\标杆班级-ansible课程大纲.assets\1570755668515.png)

  ```bash
  [root@manager project1]# cat vars_file.yml 
  web_packages: httpd
  ftp_packages: vsftpd
  
  [root@manager project1]# cat f2.yml 
  - hosts: webservers
    vars:
      - file_name: playbook_vars
  
   #调用共享vars_file文件,只不过刚好文件名叫vars_file
    vars_files: ./vars_file.yml
  
    tasks:
      - name: Create New File
        file:
          path: /tmp/{{ file_name }}
          state: touch
  
      - name: Installed Packages {{ web_packages }}
        yum:
          name: "{{ web_packages }}"
          state: present
  ```

- 2.在inventory主机清单中定义变量?

  - 1.清单文件中直接定义  hosts文件定义--

  ```bash
  [webservers]
  172.16.1.7
  172.16.1.8 
  [webservers:vars]
  file_name=hostsfile_group_vars
  ```

  - 2.创建hosts_vars  group_vars 目录

  ```bash
  [root@manager project1]# mkdir hosts_vars	#单个主机
  [root@manager project1]# mkdir group_vars	#主机组
  
  
  #1.单个主机定义和使用方式 (host_vars能分别对不同的主机定义变量)
  [root@manager project1]# cat host_vars/172.16.1.7 
  host_vars_name: 172.16.1.7
  
  [root@manager project1]# cat host_vars/172.16.1.8 
  host_vars_name: 172.16.1.8
  
  [root@manager project1]# cat f4.yml 
  - hosts: webservers
  
    tasks:
      - name: Create New File
        file:
          path: /opt/{{ host_vars_name }}
          state: touch
  
  #2.针对主机组定义的方式 
  #给指定的webservers组设定变量.其他组主机无法使用该变量
  [root@manager project1]# cat group_vars/webservers 
  group_host_vars: webservers
  
  [root@manager project1]# cat f5.yml 
  - hosts: webservers
    tasks:
      - name: Create New File {{ group_host_vars }}
        file:
          path:  /opt/{{ group_host_vars }}
          state: touch
  
  
  #3.针对主机组定义的方式  (给所有的主机和主机组设定变量)
  [root@manager project1]# cat group_vars/all 
  group_host_vars: all
  
  [root@manager project1]# cat f5.yml 
  - hosts: webservers
    tasks:
      - name: Create New File {{ group_host_vars }}
        file:
          path:  /opt/{{ group_host_vars }}
          state: touch
  ```

- 3.通过外置传参定义变量? -e

```bash
[root@manager project1]# ansible-playbook -i hosts f6.yml  -e "web_vars=123"
```

3.变量冲突,优先级?

```bash
6.定义相同的变量不同的值，来测试变量的优先级。操作步骤如下   file_name:
  1）在plabook中定义vars变量
  2）在playbook中定义vars_files变量
  3）在inventory主机定义变量
  4）在inventory主机组定义变量
  5）在host_vars中定义变量
  6）在group_vars中定义变量  组      all组
  7）通过执行命令传递变量

#1.在plabook中定义vars变量	优先级：3
[root@manager project1]# cat f2.yml 
- hosts: webservers
  vars:
    - file_name: vars
 #vars_files: ./vars_file.yml

  tasks: 
 
    - name: Create NEW File {{ file_name }}
      file: 
        path: /tmp/{{ file_name }}
        state: touch

#2.在playbook中定义vars_files变量		优先级：2
[root@manager project1]# cat vars_file.yml 
file_name: vars_file

[root@manager project1]# cat f2.yml 
- hosts: webservers
  vars:
    - file_name: vars
  vars_files: ./vars_file.yml

  tasks: 
 
    - name: Create NEW File {{ file_name }}
      file: 
        path: /tmp/{{ file_name }}
        state: touch
        
#3.在inventory主机定义变量		优先级：5
[root@manager project1]# cat hosts
[webservers]
172.16.1.7 file_name=host_172.16.1.7 
172.16.1.8 file_name=host_172.16.1.7

#4.在inventory主机组定义变量	优先级：7

#5.在host_vars中定义变量		优先级：4      
[root@manager project1]# cat hosts_vars/172.16.1.7 
file_name: host_directory_172.16.1.7

#6.在group_vars中定义变量  组	all	优先级：6
[root@manager project1]# cat group_vars/webservers 
file_name: group_vars_directory_webservers

[root@manager project1]# cat group_vars/all 
file_name: group_vars_directory_all

#7.优先级：1
[root@manager project1]# ansible-playbook -i hosts f2.yml -e "file_name=eeee"
 
优先级测试:
外置传入参数优先级最高 ---> playbook ( vars_files(共享)--->vars(私有) )  
---> host_vars  --> group_vars/group_name ---> group_vars/all
```

***4.变量注册?***

register关键字可以将某个task任务结果存储至变量中，最后使用debug输出变量内容，可以用于后续排障

```bash
[root@manager project1]# cat f8.yml 
- hosts: webservers
  tasks:
        # System_Status=$(netstat -lntp)
    - name: Get Network Status
      shell: netstat -lntp | grep "nginx"
      register: System_Status

        # echo "$System_Status"
    - name: Debug output Variables
      debug:
        msg: "{{ System_Status.stdout_lines }}"
```

***5.facts变量?***

Ansible facts 是在被管理主机上通过ansible自动采集发现的变量。facts包含每台主机特定 的主机信息。比如：被控端主机名，ip地址，系统版本，内存，磁盘状态等等。

```bash
#1.根据主机的cpu信息,生成不同的配置.
	A: 1核心    work_process 1;
	B: 2核心    work_process 2;
	
#2.根据主机名称设定不同配置文件
	zabbix_agent
		Server:   ===> 指向172.16.1.61
		Hostname:      web01   web02

#查看facts
[root@manager project1]# ansible localhost -m setup -i hosts | less

[root@manager project1]# cat ./file/zabbix_agent.conf.j2 
Server={{ zabbix_server_ip }}
ServerActive={{ zabbix_server_ip }}
Hostname={{ ansible_hostname }}

[root@manager project1]# cat f11.yml 
- hosts: webservers
  vars:
    - zabbix_server_ip: 172.16.1.61
  tasks:
    - name: Configure zabbix-agent.conf
      template:
        src: ./file/zabbix_agent.conf.j2
        dest: /tmp/zabbix-agent.conf
        
        
#3.根据主机的内存生成不同的配置文件,memcached
[root@manager project1]# cat f12.yml 
- hosts: webservers
  tasks:
    - name: Installed Memcached Server
      yum:
        name: memcached
        state: present

    - name: Configure Memcached Server
      template:
        src: ./file/memcached.j2
        dest: /etc/sysconfig/memcached
      notify: Restart Memcached Server

    - name: System Memcached Server
      systemd:
        name: memcached
        state: started
        enabled: yes

  handlers:
    - name: Restart Memcached Server
      systemd:
        name: memcached
        state: restarted

[root@manager project1]# cat file/memcached.j2 
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="{{ ansible_memtotal_mb //2 }}"
OPTIONS=""


1.根据cpu
2.根据内存
3.根据主机名
4.Redis配置文件     bind本地地址
5.操作系统不统一

		变量可以进行运算  + - * // 
		
		
		
		
#1.定义变量
	playbook
		vars			私有
		vars_files		共享
	inventory
		host_vars	
		group_vars
			group_vars/group_name
			group_vars/all
	外置传参
		-e
#2.测试优先级
	在不改变playbook变量的情况下,使用新的值测试.

#3.变量注册register
	1.将任务执行的结果存储至特定的变量中
	2.可以使用debug模块将变量进行打印输出
	
	python: 字典
	json 格式化数据
	{
        k1: v1
        k2: v2
	}
#4.facts 
```

![](E:\老男孩视频\53 老男孩教育-标杆班级-Ansible-variables-day21\标杆班级-ansible课程大纲.assets\1570768637521.png)

```bash

[root@manager project1]# cat f13.yml 
- hosts: webservers
  tasks:
    - name: RANDOM
      shell:  echo "$RANDOM"
      register: System_SJ

    - name: Debug 
      debug:
        msg: "web_{{ System_SJ.stdout }}"

#1.提取facts变量中的IP地址   mac地址  UUID 等等  只要唯一
	ansible_default_ipv4.address
[root@manager project1]# cat f14.yml 
- hosts: webservers
  tasks:

    - name: Debug 
      debug:
        msg: "web_{{ ansible_default_ipv4.address }}"
```

## Ansible 流程控制

 8.判断语句 

- 1.centos和ubuntu系统都需要安装httpd,  判断系统.
- 2.安装软件仓库,只有web组的安装webtatic其他的主机全部跳过.
- 3.TASK任务, TASK1任务执行成功,才会执行TASK2  

```bash
#根据不同的系统,安装不同的服务
- hosts: webservers
  tasks:
    - name: CentOS Installed Httpd Server
      yum:
        name: httpd
        state: present
      when: ( ansible_distribution == "CentOS" )

    - name: Ubuntu Installed Httpd Server
      yum:
        name: httpd2
        state: present
      when: ( ansible_distribution == "Ubuntu" )
      
[root@manager project1]# cat f16.yml 
- hosts: all
  tasks:
  - name: Add Nginx Yum Repository
    yum_repository:
      name: nginx
      description: Nginx Repository
      baseurl: http://nginx.org/packages/centos/7/$basearch/
    when: ( ansible_hostname is match ("web*"))


[root@manager project1]# cat f17.yml 
- hosts: webservers
  tasks:

    - name: Check Httpd Server
      command: systemctl is-active httpd
      register: Check_Httpd
      ignore_errors: yes

	#判断Check_Httpd.rc是否等于0,如果为0则执行任务,否则不执行
    - name: Restart Httpd Server
      systemd:
        name: httpd
        state: restarted
      when: ( Check_Httpd.rc == 0 )
```

9.循环语句

```bash
#一次启动多个服务
[root@manager project1]# cat f18.yml 
- hosts: webservers
  tasks:
    - name: Systemd Nginx Status
      systemd:
        name: "{{ item }}"    #调用的变量也不变,也是固定
        state: started

	#固定的语法格式
      with_items:
        - nginx
        - php-fpm


#一次拷贝多个文件
[root@manager project1]# cat f19.yml
- hosts: webservers
  tasks:
    - name: Configure nginx.conf
      copy:
        src: '{{ item.src }}'
        dest: '{{ item.dest }}'
        mode: '{{ item.mode }}'
      with_items:
        - { src: ./file/nginx.conf.j2, dest: /etc/nginx/nginx.conf, mode: '0644' }
        - { src: ./file/kold.oldxu.com.conf.j2, dest: /etc/nginx/conf.d/kold.oldxu.com.conf, mode: '0600' }



#创建多个用户,一次创建多个? 3个用户  TASK
[root@manager project1]# cat f20.yml 
- hosts: webservers
  tasks:
    - name: Create User
      user:
        name: "{{ item }}"

      with_items:
        - test1
        - test2
        - test3
        - test4


#1.创建tt1 --> bin  tt2 -->root tt3 --->adm   附加组
[root@manager project1]# cat  f20.yml 
- hosts: webservers
  tasks:
    - name: Create User
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"

      with_items:
        - { name: tt1, groups: bin }
        - { name: tt2, groups: root }
        - { name: tt3, groups: adm }
        
        
        
1.标准循环                   --->居多
	item
	with_items:
	   - test
2.字典循环:                   --->居多
    itme.name
    with_items:
        - { name: test }


3.变量循环
- hosts: webservers
  tasks:
    - name: ensure a list of packages installed
      yum: name={{ packages }} state=present
      vars:
        packages:
          - httpd
          - httpd-tools
```

10.handlers

```bash
[root@manager project1]# cat f22.yml 
- hosts: webservers
  tasks:

    - name: Installed Nginx and PHP Packages
      yum:
        name: nginx
        state: present

    - name: Configure nginx.conf 
      template:
        src: ./file/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      #监控-->changed状态-->通知-->handlers--->name-->Restart Nginx Server
      notify: Restart Nginx Server
      #notify:
      #  - Restart Nginx Server
      #  - Restart php Server

    - name: Systemd Nginx Server
      systemd:
        name: nginx
        state: started
        enabled: yes

#当nginx或php配置文件发生变更才会触发此操作
  handlers:
    - name: Restart Nginx Server
      systemd:
        name: nginx
        state: restarted


#3.handlers注意事项
	1.无论多少个task通知了相同的handlers，handlers仅会在所有tasks结束后运行一次。
	2.只有task发生改变了才会通知handlers，没有改变则不会触发handlers.
	3.不能使用handlers替代tasks、因为handlers是一个特殊的tasks。
```

变量->facts-->判断-->循环

- 1.安装Rsyncd服务  (循环)

```bash
#1.安装rsyncd
#2.配置文件，密码文件
#3.创建用户www
#4.创建目录授权

[root@manager project1]# cat rsync.yml 
- hosts: backupservers
  tasks:
  
    - name: Install Rsync
      yum:
        name: rsync
        state: present
    
    - name: Configure file
      copy: 
        src: "{{ item.src }}"
        dest: "{{ item.dest }}"
        mode: "{{ item.mode }}"
        owner: root
        group: root
      with_items:
        - { src: ./file/rsyncd.conf.j2, dest: /etc/rsyncd.conf, mode: '0644' }
        - { src: ./file/rsync.passwd.j2, dest: /etc/rsync.passwd, mode: '0600' }
      notify: System Restart Server
            
    - name: Create Group
      group: 
        name: www
        gid: 666

    - name: Create User
      user:
        name: www
        group: www
        uid: 666
        create_home: no
        shell: /sbin/nologin

    - name: Create Directory
      file: 
        path: /backup
        state: directory
        owner: www
        group: www
        recurse: yes

    - name: Open system
      systemd: 
        name: rsyncd
        state: started

       
  handlers:
    - name: System Restart Server
      systemd:
        name: rsyncd
        state: restarted
```



- 2.安装Redis   (bind  本地IP地址)    facts 

```bash
[root@manager project1]# cat redis.yml 
- hosts: backupservers
  tasks:
  
    - name: Install Redis
      yum:
        name: redis
        state: present
    
    - name: Configure file
      template: 
        src: ./file/redis.conf.j2
        dest: /etc/redis.conf
      notify: System Restart Server
            

    - name: Open system
      systemd: 
        name: redis
        state: started

       
  handlers:
    - name: System Restart Server
      systemd:
        name: redis
        state: restarted
```



- 3.安装NFS      (配置文件,创建目录,客户端挂载)    变量




