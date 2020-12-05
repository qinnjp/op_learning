## Ansible 访问控制

***1.tag标签(调试)***

​	1.对一个tasks指定一个tags标签
​	2.对一个tasks指定多个tags标签
​	3.多个tasks任务指定一个tags标签

```bash
[root@manager tasks]# cat task_nfs.yml 
- hosts: webservers
  tasks:

        #对一个任务打多个标签
    - name: Install Nfs Server
      yum: 
        name: nfs-utils
        state: present
      tags:
        - install_nfs
        - install_nfs-server


        #对一个任务打一个标签
    - name: Service Nfs Server
      service:
        name: nfs-server 
        state: started
        enabled: yes
      tags: start_nfs-server


ansible-playbook -i ../hosts  task_nfs.yml  -t start_nfs-server123
ansible-playbook -i ../hosts  task_nfs.yml  --skip-tags install_nfs-server
```

***2.include包含***

在编写playbook时，当我们发现大量的内容需要重复编写，各Tasks之间的相互功能需相互调用才能完成各自功能，或当playbook庞大到维护困难，这时我们需要使用includes。

```bash
#tasks文件没有hosts
[root@manager tasks]# cat restart_nginx.yml 
- name: Restart Nginx Server
  systemd:
    name: nginx
    state: restarted

#playbook文件
[root@manager tasks]# cat a_project.yml 
- hosts: webservers
  tasks:
    - name: A Project command
      command: echo "A"

    - name: Restart nginx
      include: ./restart_nginx.yml

[root@manager tasks]# cat b_project.yml 
- hosts: webservers
  tasks:
    - name: B Project command
      command: echo "B"

    - name: Restart nginx
      include: restart_nginx.yml

```

***3.错误处理*** ignore_errors: yes

```bash
[root@manager tasks]# cat  task_ignore.yml 
- hosts: webservers
  tasks:

#明确知道该tasks可能会报错，但不影响后续的执行
    - name: Ignore False
      command: /bin/false
      ignore_errors: yes

    - name: touch new file
      file: path=/tmp/bgx_ignore state=touch
      

```

***3.异常处理***

​	1.每次状态都是changed,纵使没有修改过被控端

```bash
        #Check_Redis_Status=$(netstat -lntp | grep redis) 
    - name: Check Redis Status
      shell: netstat -lntp | grep redis
      register: Check_Redis_Status
      changed_when: false

        #echo ${Check_Redis_Status}
    - name: Debug Check_Redis Variables
      debug:
        msg: "Redis Status: {{ Check_Redis_Status.stdout_lines }}" 
```

​	2.nginx推送配置配置文件后没有任何检查功能

```bash
###########################################low版

[root@manager tasks]# cat task_nginx.yml 
- hosts: webservers
  tasks:

        #安装nginx
    - name: Installed nginx Server
      yum: 
        name: nginx
        state: present

        #配置nginx
    - name: Configure nginx Server
      template:
        src: ./file/nginx.conf.j2
        dest: /etc/nginx/nginx.conf


        #检查nginx (Check_Nginx_Status=$(nginx -t))
    - name: Check Nginx Configure File
      shell: nginx -t
      register: Check_Nginx_Status
      ignore_errors: yes
      changed_when: false

        #启动Nginx
    - name: Systemd Nginx Server
      systemd:
        name: nginx
        state: started
        enabled: yes 
      when: ( Check_Nginx_Status.rc == 0 )  #只有Check_Nginx_Status.rc=0时,才会执行启动操作
      
      
##################################new版
[root@manager tasks]# cat task_nginx.yml 
- hosts: webservers
  tasks:
        #安装nginx
    - name: Installed nginx Server
      yum: 
        name: nginx
        state: present

        #配置nginx
    - name: Configure nginx Server
      template:
        src: ./file/nginx.conf.j2
        dest: /etc/nginx/nginx.conf


        #检查nginx (Check_Nginx_Status=$(nginx -t))
    - name: Check Nginx Configure File
      shell: nginx -t
      register: Check_Nginx_Status
      changed_when: 
        - Check_Nginx_Status.stdout.find('successful')
        - false
   
        #启动Nginx
    - name: Systemd Nginx Server
      systemd:
        name: nginx
        state: started
        enabled: yes
        
##################################new版       
[root@manager tasks]# cat task_nginx.yml 
- hosts: webservers
  tasks:
        #安装nginx
    - name: Installed nginx Server
      yum: 
        name: nginx
        state: present

        #配置nginx
    - name: Configure nginx Server
      template:
        src: ./file/nginx.conf.j2
        dest: /etc/nginx/nginx.conf


        #检查nginx (Check_Nginx_Status=$(nginx -t))
    - name: Check Nginx Configure File
      shell: nginx -t
   
        #启动Nginx
    - name: Systemd Nginx Server
      systemd:
        name: nginx
        state: started
        enabled: yes      
```

​	3.强制调用handlers触发器

```bash
[root@manager tasks]# cat task_nginx.yml 
- hosts: webservers
  force_handlers: yes	#无论tasks失败与否,只要通过过handlers,那me一定会执行
  tasks:
        #安装nginx
    - name: Installed nginx Server
      yum: 
        name: nginx
        state: present

        #配置nginx
    - name: Configure nginx Server
      template:
        src: ./file/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx Server

        #检查nginx (Check_Nginx_Status=$(nginx -t))
    - name: Check Nginx Configure File
      shell: nginx -t

        #启动Nginx
    - name: Systemd Nginx Server
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

总结: Ansible Task 控制

- 1.判断语句  when
- 2.循环语句  with_items
- 3.触发器  handlers
- 4.标签   tag
- 5.忽略错误   ignore_errors
- 6.异常处理
  - 1.关闭TASK的changed状态  -->让该TASK一直处理OK状态  changed_when: false
  - 2.强制调用handlers:  当正常task调用过handlers,则无论后续的task成功还是失败,都会调用handlers触发器执行任务.
  - 3........ Check_Nginx_Status.stdout.find('successful')



## Ansible Jinja2 模板

1.jinja2渲染NginxProxy配置文件

- **jinja2**
  - 房屋建筑设计固定的?
- **jinja2模板与Ansible关系**
- **Ansible如何使用jinja2模板**
  - template模块     拷贝文件?
  - template copy  区别?  
    - template会解析配置文件中的变量
    - copy  不会解析任何的变量,只会拷贝文件

*Ansible允许jinja2模板中使用判断  循环，但是jinja判断循环语法不允许在playbook中使用。*

*注意: 不是每个管理员都需要这个特性，但是有些时候jinja2模板能大大提高效率。*



**1.jinja模板基本语法**
*1）要想在配置文件中使用jinj2，playbook中的tasks 必须使用template模块*

*2）模板配置文件里面使用变量，比如 {{ PORT }} 或使用 {{ facts 变量 }}*

**2.jinja模板逻辑关系**
*{% for i in EXPR %}...{% endfor%} 作为循环表达式*

*{% if EXPR %}...{% elif EXPR %}...{% endif%} 作为条件判断*

*{# COMMENT #} 表示注释*

```bash
{% for i in range(1,10)%}
        server 172.16.1.{{i}};
{% endfor %}


#判断
{% if ansible_fqdn == "web01" %}
        echo 123
{% elif ansible_fqdn == "web02" %}
        echo 456
{% else %}
        echo 789
{% endif %}
```

nginxproxy配置文件

```bash
[root@manager jinja2]# cat j_nginx.yml 
- hosts: lbservers
  tasks:

        #安装nginx
    - name: Installed nginx Server
      yum: 
        name: nginx
        state: present

        #配置nginx vhosts
    - name: Configure nginx Server
      template:
        src: ./file/proxy_kod.oldxu.com.conf.j2
        dest: /etc/nginx/conf.d/proxy_kod.oldxu.com.conf
      notify: Restart Nginx Server


        #启动Nginx
    - name: Systemd Nginx Server
      systemd:
        name: nginx
        state: started
        enabled: yes 


  handlers:
    - name: Restart Nginx Server
      systemd: 
        name: nginx
        state: restarted
        
        
# nginx组变量   
[root@manager jinja2]# cat group_vars/all 
kod_http_port: 80
kod_server_name: kod.oldxu.com
kod_web_site: /code/kod

 

#nginx proxy配置文件渲染
[root@manager jinja2]# cat file/proxy_kod.oldxu.com.conf.j2 
upstream {{ kod_server_name }} {
    {% for host in groups['webservers'] %}
	server {{host}}:{{kod_http_port}};
    {% endfor %}
}

server {
	listen {{ kod_http_port }};
	server_name  {{ kod_server_name }};

	location / {
		proxy_pass http://{{ kod_server_name }};
		proxy_set_header Host $http_hosts;
	}
}

[root@manager jinja2]# cat ../hosts
[webservers]
172.16.1.7
172.16.1.8
```

2.Keepalived配置文件   master   slave

​	1.准备多个配置文件   master   backup

```bash
[root@manager jinja2]# cat j_keepalived.yml 
- hosts: lbservers
  tasks:
    - name: Installed Keepalived Server
      yum:
        name: keepalived
        state: present

    - name: Configure Keepalived Master
      copy:
        src: ./file/keepalived-master.conf.j2
        dest: /etc/keepalived/keepalived.conf
      when: ( ansible_hostname == "lb01" )
      notify: Restart Keepalived Server

    - name: Configure Keepalived Backup
      copy:
        src: ./file/keepalived-backup.conf.j2
        dest: /etc/keepalived/keepalived.conf
      when: ( ansible_hostname == "lb02" )
      notify: Restart Keepalived Server

    - name: Systemd Keepalived Server
      systemd:
        name: keepalived
        state: started
        enabled: yes

  handlers:
    - name: Restart Keepalived Server
      systemd:
        name: keepalived
        state: restarted
```

​	2.设定host_vars变量  5和6设定相同的变量,不同的值

```bash
#1.准备一份keepalived配置文件
#2.需要在keepalived配置文件中使用变量方式  ---> jinja

[root@manager jinja2]# cat ./file/keepalived-vars.conf.j2 
global_defs {     
    router_id {{ ansible_hostname }}
}

vrrp_instance VI_1 {
    state  {{ state }}
    priority {{ priority }}

    interface eth0
    virtual_router_id 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
}
    virtual_ipaddress {
        10.0.0.3
    }
}



[root@manager jinja2]# cat host_vars/172.16.1.5
state: MASTER
priority: 200
[root@manager jinja2]# cat host_vars/172.16.1.6
state: BACKUP
priority: 99

[root@manager jinja2]# cat var_keepalived.yml 
- hosts: lbservers
  tasks:

    - name: Installed Keepalived Server
      yum:
        name: keepalived
        state: present


    - name: Configure Keepalived Master
      template:
        src: ./file/keepalived-vars.conf.j2
        dest: /etc/keepalived/keepalived.conf
      notify: Restart Keepalived Server

    - name: Systemd Keepalived Server
      systemd:
        name: keepalived
        state: started
        enabled: yes

  handlers:
    - name: Restart Keepalived Server
      systemd:
        name: keepalived
        state: restarted
```

​	3.jinja2判断方式

```bash
[root@manager jinja2]# cat jinja_keepalived.yml 
- hosts: lbservers
  tasks:

    - name: Installed Keepalived Server
      yum:
        name: keepalived
        state: present


    - name: Configure Keepalived Master
      template:
        src: ./file/keepalived.conf.j2
        dest: /etc/keepalived/keepalived.conf
      notify: Restart Keepalived Server

    - name: Systemd Keepalived Server
      systemd:
        name: keepalived
        state: started
        enabled: yes

  handlers:
    - name: Restart Keepalived Server
      systemd:
        name: keepalived
        state: restarted


[root@manager jinja2]# cat file/keepalived.conf.j2 
global_defs {     
    router_id {{ ansible_hostname }}
}

vrrp_instance VI_1 {
{% if ansible_hostname == "lb01" %}
    state  MASTER
    priority 150
{% elif ansible_hostname == "lb02" %}
    state  BACKUP
    priority 100
{% endif %}
#########################相同的内容
    interface eth0
    virtual_router_id 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
}
    virtual_ipaddress {
        10.0.0.3
    }
}
```

## Ansible Roles角色

*Roles小技巧:*

1.创建roles目录结构，手动或使用ansible-galaxy init test roles

2.编写roles的功能，也就是tasks。  nginx  rsyncd memcached

3.最后playbook引用roles编写好的tasks

~~~bash
Ansible Roles角色
mkdir /root/roles/nginx/{tasks,templates,handlers}

##tasks
[root@manager ~]# cat /root/roles/nginx/tasks/main.yml 
- name: Install Nginx Server
  yum:
    name: nginx
    state: present

- name: Configure Nginx Server
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart Nginx Server


- name: Systemd Nginx Server
  systemd:
    name: nginx
    state: started
    enabled: yes

##template
[root@manager roles]# cat /root/roles/nginx/templates/nginx.conf.j2 
user www;
worker_processes  {{ ansible_processor_vcpus }};

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  {{ ansible_processor_vcpus * 1024 }};
}
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
	access_log /var/log/nginx/access.log main;

    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    #gzip  on;
    include /etc/nginx/conf.d/*.conf;
}

###handlers
[root@manager ~]# cat /root/roles/nginx/handlers/main.yml 
- name: Restart Nginx Server
  systemd:
    name: nginx
    state: restarted
    
    
    
#调用playbook
[root@manager roles]# cat /root/roles/site.yml 
- hosts: webservers
  roles:
    - nginx

##hosts ansible.cfg  自备
~~~

*memcached roles*

```bash
#安装
#配置
#启动

#1.创建roles的目录结构
[root@manager roles]# mkdir memcached/{tasks,templates,handlers} -p

#2.编写对应的tasks  (1.安装  2配置(templates)  3.启动  4.重启(handlers) )
[root@manager roles]# cat memcached/tasks/main.yml 
- name: Installed Memecached Server
  yum:
    name: memcached
    state: present

- name: Configure Memcached Server
  template:
    src: memcached.j2
    dest: /etc/sysconfig/memcached
  notify: Restart Memcached Server


- name: System Memcached Server
  systemd:
    name: memcached
    state: started
    enabled: yes

[root@manager roles]# cat memcached/templates/memcached.j2 
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="{{ ansible_memtotal_mb //2 }}"
OPTIONS=""

[root@manager roles]# cat memcached/handlers/main.yml 
- name: Restart Memcached Server
  systemd:
    name: memcached
    state: restarted


#3.playbook调用roles
[root@manager roles]# cat site.yml 
- hosts: webservers
  roles:
    - { role: nginx, tags: web }
    - { role: memcached, tags: cache }
```

*NFS服务*

```bash
#1.创建项目目录结构   ---> 
[root@manager roles]# mkdir nfs/{tasks,templates,handlers} -p

#2.编写task任务



#3.playbook调用roles项目
```

- roles:
  - 1.nginxProxy+keepalived  10.0.0.5  10.0.0.6      10.0.0.3
  - 2.nginx静态网站                             172.16.1.7 172.16.1.8

