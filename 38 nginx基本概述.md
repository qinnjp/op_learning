 

# nginx

**1.介绍nginx**

- nginx是一个开源且高性能,可靠的Http Web服务,代理服务.
  - 开源,体现在直接获取nginx的源代码
  - 高性能,体现在支持海量的并发
  - 高可靠,体现在服务稳定

**2.为什么选择nginx**

- 高性能,高并发
- 高扩展性
- 高可靠性
- 热部署

![1568632510804](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1568632510804.png)

- 互联网公司都选择nginx,具备企业最常使用的功能,其次nginx同一技术降低维护成本,同时降低技术成本。
- nginx使用Epool网络管理而常听到的Apache使用Select网络模型。
  - Select：当用户发起一次请求，select进行遍历扫描，从而使性能低下。
  - Epool：当用户发起请求，Epool模型会直接进行处理，效率高效。

http://www.cnblogs.com/zm-0713/p/5064168.html

**3.nginx的使用场景**

- 反向代理
- 负载均衡
- 代理缓存
- 静态资源
- 动静分离
- Https

**4.nginx安装 配置 启动**

- 第一种：源码安装
- 第二种：yum  ---> 官方仓库   版本新
- 第三种：yum  --->epel源     版本旧

 

-  1.安装官方仓库源

  ~~~bash
  [root@web01 ~]# cat /etc/yum.repos.d/nginx.repo 
  [nginx-stable]
  name=nginx stable repo
  baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
  gpgcheck=1
  enabled=1
  gpgkey=https://nginx.org/keys/nginx_signing.key
  ~~~

- 2.使用yum直接安装

  ~~~bash
  [root@web01 ~]# yum install nginx -y
  ~~~

- 3.启动nginx

- Nginx 配置 了解

  ~~~bash
  [root@web01 ~]# cat /etc/nginx/nginx.conf
  
  user  nginx;									# nginx进程的用户身份
  worker_processes  1;							# nginx的工作进程数量
  error_log  /var/log/nginx/error.log warn;		# 错误日志的路径 [警告级别才会记录]
  pid        /var/run/nginx.pid;					# 进程运行后,会产生一个pid
  
  
  events {										# 事件模型
      worker_connections  1024;					# 每个work能够支持的连接数
  	use epoll;									# 使用epoll网络模型
  }
  
  
  http {											# 接收用户的http请求
      include       /etc/nginx/mime.types;		# 包含所有静态资源的文件
      default_type  application/octet-stream;		# 默认类型 (下载)
  
  	日志相关
      log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';
  
      access_log  /var/log/nginx/access.log  main;	# 访问日志的路径
      #sendfile        on;
      #tcp_nopush     on;
      keepalive_timeout  65;		#长链接超时时间
      #gzip  on;					#启用压缩功能
  	
  	
  	#使用Server配置网站, 每个Server{}代表一个网站
  	server {
  		listen 80;
  		server_name test.oldxu.com;
  		
  		location / {					#控制网站访问的路径
  			root ...;
  		}
  	}
  
      include /etc/nginx/conf.d/*.conf;		包含哪些文件
  }
  ~~~

  PS：nginx中的http、server，location之间的关系是？

  - http：标签主要用来解决用户的请求与响应。
  - server： 标签主要用来响应具体的某一个网站。
  - location：标签主要用于匹配网站具体的url路径。



​	http{} 层下允许有多个server{} ，可以有多个网站。

​	一个server{} 下又允许有多个location{} 每个网站的uri路径不同，所以要分别进行匹配。

**6.Nginx 搭建 游戏网站**

- 1.注释掉之前的默认网站

  ~~~bash
  [root@web01 html]# cd /etc/nginx/conf.d/
  [root@web01 conf.d]# gzip default.conf 
  ~~~

- 2.编写游戏网站Nginx配置文件

  ~~~bash
  [root@web01 conf.d]# cat game.oldxu.com.conf 
  server {
  	listen 80;			#该网站提供访问的端口
  	server_name game.oldxu.com;	#访问该网站的域名
  	
  	location / {
  	root /code;
  	index index.html;
  }
  }
  ~~~

- 3.根据Nginx的配置文件,初始化

  ~~~bash
  [root@web01 conf.d]# mkdir /code
  ~~~

- 4.上传代码

  ~~~bash
  [root@web01 conf.d]# cd /code/
  [root@web01 code]# rz html5.zip
  [root@web01 code]# unzip html5.zip 
  ~~~

- 5.检测语法

  ~~~bash
  [root@web01 code]# nginx -t
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etc/nginx/nginx.conf test is successful
  ~~~

- 6.重载服务

  ~~~bash
  [root@web01 code]# systemctl restart nginx
  ~~~

- 7.配置域名解析

- 8.Nginx访问的整体流程

  http://    game.oldxu.com     /      game/yibihua/index.html

  请求的uri:		/game/yibihua/index.html
  真实映射位置:   /code/game/yibihua/index.html

- 9.Nginx 搭建 多个游戏网站  ---> 虚拟主机

  虚拟主机:  在一台服务器上运行多套网站



Nginx配置虚拟主机有如下三种方式：
	方式一、基于主机多IP方式					10.0.0.7  172.16.1.7
	方式二、基于端口的配置方式					80 81 82 83
	方式三、基于名称方式(多域名方式)			test1 test2 test3		<---推荐



- 方式一、基于主机多IP方式

  ~~~bash
  [root@web01 conf.d]# cat ip_eth0.conf 
  server {
  	listen 10.0.0.7:80;
  	location / {
  		root /ip1;
  		index index.html;
  	}
  }
  server {
  	listen 172.16.1.7:80;
  	location / {
  		root /ip2;
  		index index.html;
  	}
  }
  [root@web01 conf.d]# mkdir /ip1 /ip2
  [root@web01 conf.d]# echo "10...." > /ip1/index.html
  [root@web01 conf.d]# echo "172...." > /ip2/index.html
  [root@web01 conf.d]# systemctl restart nginx
  
  
  # 测试访问
  [root@web01 ~]# curl http://10.0.0.7
  10.... 
  [root@web01 ~]# curl http://172.16.1.7
  ~~~

- 172....方式二、基于端口的配置方式	  81 82 83
  	公司内部有多套系统,希望部署在一台服务器上, 而内网又没有域名.
    	所以,我们可以通过相同IP,不同的端口,访问不同的网站页面.

  ~~~bash
  [root@web01 conf.d]# cat port.conf 
  server {
  	listen 81;
  
  	location / {
  		root /81;
  		index index.html;
  	}
  }
  
  server {
  	listen 82;
  
  	location / {
  		root /82;
  		index index.html;
  	}
  }
  
  server {
  	listen 83;
  
  	location / {
  		root /83;
  		index index.html;
  	}
  }
  [root@web01 conf.d]# mkdir /81 /82 /83
  [root@web01 conf.d]# echo "81" > /81/index.html
  [root@web01 conf.d]# echo "82" > /82/index.html
  [root@web01 conf.d]# echo "83" > /83/index.html
  ~~~

  

- 三个网站运行在同一台服务器,只需要通过不同的域名来实现访问:
  		game
    		wzq
    		tk

io网络模型:
	同步
	异步
	阻塞
	非阻塞
	同步阻塞
	同步非阻塞
	异步阻塞
	异步非阻塞

https://www.jianshu.com/p/d4f24abc8024?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation