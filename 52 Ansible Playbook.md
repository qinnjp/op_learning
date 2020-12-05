## Ansible Playbook

- 1.什么是playbook？

  - playbook	（剧本） 定义一个文本文件，以yml为后缀结尾

    - play  定义主机的角色

    - tasks 定义具体执行的任务 

      总结：playbook是由一个或多个play组成，一个play可以包含多个task任务。

      可以理解为使用不同的模块来完成共同的事情。

- 2.playbook和ad-hoc的区别？

  playbook和ad-hoc的关系

  - playbook是对ad-hoc的一种编排方式。
  - playbook可以持久运行，而ad-hoc只能临时7运行。
  - playbook是和复杂的任务，而ad-hoc适合做快速简单的任务。
  - playbook能控制任务执行的先后顺序。

- 3.Playbook三板斧？缩进 冒号 短横线 （语法格式）

  playbook是由yml语法书写，结构清晰，可读性强，所以必须掌握yml基础语法

| 语法   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| 缩进   | yaml使用固定的缩进风格表示层级结构，每个缩进有两个空格组成，不能使用tabs |
| 冒号   | 以冒号结尾的除外，其他所有冒号后面必须有空格                 |
| 短横线 | 表示列表项，使用一个短横杠加一个空格，多个项使用同样的缩进级别作为同一列表 |

- 4.playbook写服务 （NFS Httpd Nginx LAMP）

***案例一、使用ansible安装并配置nfs服务***

```bash
#1.安装nfs服务
#2.编写配置文件
#3.创建组和用户
#4.建立共享目录授权
#6.启动服务
#7.检测配置文件是否修改，若修改重启服务
#8.客户端挂载

#project
#配置公钥私钥
[root@manager ~]# ssh-keygen -C manager@qq.com	#配置公钥私钥 一路回车
[root@manager ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.16.1.31
[root@manager ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.16.1.41

#配置用户清单
[root@manager project1]# cat hosts 
[nfsservers]
172.16.1.31

[backupservers]
172.16.1.41

[web:children]
nfsservers
backupservers

[webservers]
172.16.1.7 
172.16.1.8

#服务端配置
[root@manager project1]# cat nfs_server.yml 
- hosts: nfsservers
  tasks: 

    - name: Install NFS Server
      yum:
        name: nfs-utils
        state: present

    - name: Create NFS Profile
      copy:
        src: ./file/exports.j2
        dest: /etc/exports
        owner: root
        group: root
        mode: 0644
      notify: Restart NFS Server

    - name: Create Group
      group:
        name: www
        gid: 666

    - name: Create User
      user: 
        name: www
        uid: 666
        group: www
        create_home: no
        shell: /sbin/nologin

    - name: Create NFS Directory
      file: 
        path: /ansible_data
        owner: www
        group: www
        state: directory
        recurse: yes

    - name: Start Systemd
      systemd:
        state: started
        name: nfs

  handlers:
    - name: Restart NFS Server
      systemd: 
        name: nfs
        state: restarted
  
#运行
[root@manager project1]# ansible-playbook nfs_server.yml -i hosts

#客户端配置
[root@manager project1]# cat nfs_client.yml 
- hosts: webservers
  tasks:

    - name: Mount 
      mount: 
        path: /mnt
        src: 172.16.1.31:/ansible_data
        fstype: nfs
        opts: defaults
        state: mounted
#运行
[root@manager project1]# ansible-playbook nfs_client.yml -i hosts

#检查
[root@web01 nginx]# df -h
172.16.1.31:/ansible_data   38G  1.6G   37G   5% /mnt

```

***案例二、使用ansible安装并配置nginx服务***

```bash
1.安装		yum
2.配置		copy
3.启动		systmd
handlers

[root@manager project1]# cat nginx.yml 
- hosts: webservers
  tasks:

    - name: Installed Nginx Server
      yum:
        name: nginx
        state: present

    - name: Configure Nginx Server
      copy:
        src: ./file/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: 0644
        backup: yes
      notify: Restart Nginx Server
      
    - name: Systmd nginx Server
      systemd:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Restart Nginx Server
      systemd:
        name: nginx
        state: restarted
```

***案例三、使用AnsiblePlaybook方式构建LAP架构，具体操作步骤如下 :***

1.使用yum安装 httpd、php、firewalld等   7.1   5.3

2.使用get_url下载http://fj.xuliangwei.com/public/index.php文件

3.启动httpd、firewalld、等服务

4.添加防火墙规则，放行http的流量

```bash
#hosts配置
[root@manager project1]# cat hosts 
[nfsservers]
172.16.1.31

[backupservers]
172.16.1.41

[web:children]
nfsservers
backupservers

[webservers]
172.16.1.7
172.16.1.8

#具体配置
[root@manager project1]# cat lamp.yml 
- hosts: web
  tasks:
    - name: Installed Httpd Server
      yum: 
        name: httpd
        state: present

    - name: Installed PHP Server
      yum: 
        name: php
        state: present

    - name: Configure Httpd WebSite
      get_url:
        url: http://fj.xuliangwei.com/public/index.php
        dest: /var/www/html/index.php
        mode: 0644

    - name: Systemd Httpd Server
      systemd:
        name: httpd
        state: started

    - name: Systemd Firewalld Server
      systemd:
        name: firewalld
        state: started


    - name: Configure Firewalld Rule
      firewalld:
        service: http
        state: enabled
```

![](E:\老男孩视频\53 老男孩教育-标杆班级-Ansible-variables-day21\标杆班级-ansible课程大纲.assets\1570680036415.png)

***案例五、搭建可道云网盘   31   41    apache+php***

1.安装      apache+php
2,下载代码
3.启动      systemd
4.下载代码   wget  解压

- 作业:  Nginx+PHP 搭建可道云

- 1.先手动实现
  - 1.配置yum源     nginx  php
  - 2.安装软件包    (循环的方式)
    - nginx  php71w
  - 3.创建用户      www  统一UID和GID
  - 4.配置nginx.conf配置文件,修改启用用户为www
  - 5.配置php的权限  /etc/php-fpm.d/www.conf
  - 6.添加虚拟主机  /etc/nginx/conf.d/xx.conf
  - 7.创建网站的站点目录
  - 8.传输代码至站点目录
  - 9.启动nginx和php
  - 10.修改配置还需要能够实现自动重启
- 2.ansible方式
- 推代码 (git+jenkins)
  - 1.如果是文件夹, 如何防止重复推送
  - 2.如果是压缩包,又怎么办呢?    

```bash
- hosts: web
  tasks:

     #1.配置yum源仓库 nginx php
    - name: Installed Nginx repo
      yum_repository:
        name: nginx
        description: nginx repos
        baseurl: http://nginx.org/packages/centos/$releasever/$basearch/
        gpgcheck: no

     #2.配置yum源仓库 php
    - name: Installed PHP repo
      yum_repository:
        name: webtatic-php
        description: php repos
        baseurl: http://us-east.repo.webtatic.com/yum/el7/x86_64/ 
        gpgcheck: no

    #3.安装nginx和php
    - name: Installed Nginx and PHP Packages
      yum:
        name: "{{ packages }}"
      vars:
        packages: 
          - nginx
          - php71w
          - php71w-cli
          - php71w-common
          - php71w-devel
          - php71w-gd
          - mod_php71w
          - php71w-fpm
          - php71w-opcache

    #4.创建程序启动的用户身份
    - name: Create Group www
      group:
        name: www
        gid: 666

    - name: Create User www
      user:
        name: www
        group: www
        uid: 666
        create_home: no
        shell: /sbin/nologin

     #5.管理nginx配置文件
    - name: Configure nginx.conf 
      copy:
        src: ./file/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx Server
     
     #6.管理php-fpm配置文件
    - name: Configure php-fpm.conf
      copy:
        src: ./file/php-www.conf.j2
        dest: /etc/php-fpm.d/www.conf
      notify: Restart PHP-FPM Server

     #6.添加kodcloud虚拟主机(检测语法)
    - name: Add Nginx VirtHost kod.oldxu.com
      copy:
        src: ./file/kold.oldxu.com.conf.j2
        dest: /etc/nginx/conf.d/kold.oldxu.com.conf
      notify: Restart Nginx Server

    - name: Init Nginx BseEnv
      file:
        path: /code
        state: directory
        owner: www
        group: www
        recurse: yes

    - name: Push KodCloud Code
      synchronize:
        src: ./file/kod
        dest: /code/

    - name: Chomod kodcloud
      file:
        path: /code
        owner: www
        group: www
        mode: 0777
        recurse: yes

    - name: Systemd Nginx Server
      systemd:
        name: nginx
        state: started
        enabled: yes

    - name: Systemd PHP-FPM Server
      systemd:
        name: php-fpm
        state: started
        enabled: yes
        

#当nginx或php配置文件发生变更才会触发此操作
  handlers:
    - name: Restart Nginx Server
      systemd:
        name: nginx
        state: restarted

    - name: Restart PHP-FPM Server
      systemd:
        name: php-fpm
        state: restarted

```

