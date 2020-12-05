# nginx代理与负载均衡



## 正向代理与反向代理

**1.什么是代理**

​	代为办理

![img](https://img2018.cnblogs.com/blog/1488697/201905/1488697-20190529080903285-1700207365.png)

**2.nginx正向代理，反向代理？**

- 1.那Nginx作为代理服务, 按照应用场景模式进行总结，代理分为正向代理、反向代理

  正向代理，(内部上网) 客户端<-->代理->服务端

![img](https://img2018.cnblogs.com/blog/1488697/201905/1488697-20190529081130465-400587736.png)

​		反向代理，用于公司集群架构中，客户端->代理<-->服务端

![img](https://img2018.cnblogs.com/blog/1488697/201905/1488697-20190529081139288-1983258210.png)

- 2.正向与反向代理的区别
  区别在于形式上服务的"对象"不一样
  正向代理代理的对象是客户端，为客户端服务
  反向代理代理的对象是服务端，为服务端服务

**3.Nginx代理支持哪些协议、常用的是哪些?**

- 1.Nginx作为代理服务，可支持的代理协议非常的多，具体如下图

![img](https://img2018.cnblogs.com/blog/1488697/201905/1488697-20190529081228829-1540914875.png)

- 2.如果将Nginx作为反向代理服务，常常会用到如下几种代理协议，如下图所示

![img](https://img2018.cnblogs.com/blog/1488697/201905/1488697-20190529081239080-1861577771.png)

- 3.反向代理模式与Nginx代理模块总结如表格所示

| **反向代理模式**       | **Nginx配置模块**       |
| ---------------------- | ----------------------- |
| http、websocket、https | ngx_http_proxy_module   |
| fastcgi                | ngx_http_fastcgi_module |
| uwsgi                  | ngx_http_uwsgi_module   |
| grpc                   | ngx_http_v2_module      |

**4.nginx反向代理配置语法**

- 1.nginx代理配置语法

~~~nginx
Syntax: proxy_pass URL;
Default:    —
Context:    location, if in location, limit_except

http://localhost:8000/uri/
http://192.168.56.11:8000/uri/
http://unix:/tmp/backend.socket:/uri/
~~~

- 2.url跳转修改返回Location[不常用]参考URL

~~~nginx
Syntax: proxy_redirect default;
proxy_redirect off;proxy_redirect redirect replacement;
Default:    proxy_redirect default;
Context:    http, server, location
~~~

- 3.添加发往后端服务器的请求头信息

~~~nginx
Syntax: proxy_set_header field value;
Default:    proxy_set_header Host $proxy_host;
            proxy_set_header Connection close;
Context:    http, server, location

# 用户请求的时候HOST的值是www.bgx.com, 那么代理服务会像后端传递请求的还是www.bgx.com
proxy_set_header Host $http_host;
# 将$remote_addr的值放进变量X-Real-IP中，$remote_addr的值为客户端的ip
proxy_set_header X-Real-IP $remote_addr;
# 客户端通过代理服务访问后端服务, 后端服务通过该变量会记录真实客户端地址
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
~~~

- 4.代理到后端的TCP连接、响应、返回等超时时间

~~~nginx
//nginx代理与后端服务器连接超时时间(代理连接超时)
Syntax: proxy_connect_timeout time;
Default: proxy_connect_timeout 60s;
Context: http, server, location

//nginx代理等待后端服务器的响应时间
Syntax: proxy_read_timeout time;
Default:    proxy_read_timeout 60s;
Context:    http, server, location

//后端服务器数据回传给nginx代理超时时间
Syntax: proxy_send_timeout time;
Default: proxy_send_timeout 60s;
Context: http, server, location
~~~

- 5.proxy_buffer代理缓冲区

~~~nginx
//nignx会把后端返回的内容先放到缓冲区当中，然后再返回给客户端,边收边传, 不是全部接收完再传给客户端
Syntax: proxy_buffering on | off;
Default: proxy_buffering on;
Context: http, server, location

//设置nginx代理保存用户头信息的缓冲区大小
Syntax: proxy_buffer_size size;
Default: proxy_buffer_size 4k|8k;
Context: http, server, location

//proxy_buffers 缓冲区
Syntax: proxy_buffers number size;
Default: proxy_buffers 8 4k|8k;
Context: http, server, location
~~~

- 6.Proxy代理网站常用优化配置如下，将配置写入新文件，调用时使用include引用即可

~~~nginx
[root@Nginx ~]# vim /etc/nginx/proxy_params
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

proxy_connect_timeout 30;
proxy_send_timeout 60;
proxy_read_timeout 60;

proxy_buffering on;
proxy_buffer_size 32k;
proxy_buffers 4 128k;
~~~

- 7.代理配置location时调用, 方便后续多个Location重复使用

~~~nginx
location / {
    proxy_pass http://127.0.0.1:8080;
    include proxy_params;
}
~~~

![1569046892972](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1569046892972.png)

- 8.反向代理示例

  代理配置

~~~bash
[root@lb01 conf.d]# cat proxy_web.oldxu.com.conf 
server {
	listen 80;
	server_name web.oldxu.com;

	location / {
		proxy_pass http://10.0.0.7:80;
		include proxy_params;
	}
}
~~~

​	优化配置文件

~~~bash
[root@lb01 conf.d]# cat /etc/nginx/proxy_params 
proxy_http_version 1.1;   #向后端请求http1.1协议
proxy_set_header Host $http_host;  #发往后端web服务器的头信息
proxy_set_header X-Real-IP $remote_addr;	#	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;	#透传真实ip

proxy_connect_timeout 30;    # 连接超时时间
proxy_send_timeout 60;	#发送超时时间
proxy_read_timeout 60;	#读取超时时间

proxy_buffering on;
proxy_buffer_size 32k;
proxy_buffers 4 128k;
~~~

​	后端web配置

~~~bash
[root@web01 conf.d]# cat web.oldxu.com.conf 
server {
	listen 80;
	server_name web.oldxu.com;

	location / {
		root /html;
		index index.html;
	}
}
~~~

## 负载均衡

**1.什么是负载均衡**

​	负载均衡Load Balance，其含义就是将负载（工作任务）进行平衡，分摊到多个操作单元上进行运行。

**2.为什么要使用负载均衡**

​	当我们的Web服务器直接面向用户，往往要承载大量并发请求，单台服务器难以负荷，我使用多台WEB服务器组成集群，前端使用Nginx负载均衡，将请求分散的打到我们的后端服务器集群中，实现负载的分发。那么会大大提升系统的吞吐率、请求性能、高容灾。

![img](https://img2018.cnblogs.com/blog/1488697/201906/1488697-20190603202411175-869933088.png)

**3.负载均衡能实现的应用场景一: 四层负载均衡**

​	所谓四层负载均衡指的是OSI七层模型中的传输层，那么传输层Nginx已经能支持TCP/IP的控制，所以只需要对客户端的请求进行TCP/IP协议的包转发就可以实现负载均衡，那么它的好处是性能非常快、只需要底层进行应用处理，而不需要进行一些复杂的逻辑。

![img](https://img2018.cnblogs.com/blog/1488697/201906/1488697-20190603202720540-81577200.png)

**4.负载均衡能实现的应用场景二: 七层负载均衡**

![img](https://img2018.cnblogs.com/blog/1488697/201906/1488697-20190603202740961-908793897.png)

> ## 四层负载均衡与七层负载均衡区别
>
> 四层负载均衡数据包在底层就进行了分发，而七层负载均衡数据包则是在最顶层进行分发、由此可以看出，七层负载均衡效率没有四负载均衡高。
> 但七层负载均衡更贴近于服务，如:http协议就是七层协议，我们可以用Nginx可以作会话保持，URL路径规则匹配、head头改写等等，这些是四层负载均衡无法实现的。

**5.七层负载均衡配置示例?**

~~~~bash
[root@lb01 conf.d]# cat proxy_web.oldxu.com.conf 
upstream web {
	server 172.16.1.7:80;
	server 172.16.1.8:80;
	}

server {
	listen 80;
	server_name web.oldxu.com;
	location / {
	proxy_pass http://web;
	include proxy_params;
	}
}

[root@lb01 conf.d]# cat /etc/nginx/proxy_params 
proxy_http_version 1.1;
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

proxy_connect_timeout 30;
proxy_send_timeout 60;
proxy_read_timeout 60;

proxy_buffering on;
proxy_buffer_size 32k;
proxy_buffers 4 128k;

#后端web配置 (为了区分,将两台web的站点配置的不一样,以便测试效果)
[root@web01 conf.d]# cat web.oldxu.com.conf 
server {
	listen 80;
	server_name web.oldxu.com;
#后端web配置 (为了区分,将两台web的站点配置的不一样,以便测试效果)
[root@web01 conf.d]# cat web.oldxu.com.conf 
server {
	listen 80;
	server_name web.oldxu.com;
	location / {
		root /html;
		index index.html;
	}
	}
~~~~

**6.七层负载均衡整合集群架构示例?**	

​	将示例中的替换为

​	blog
​	zh

**7.Nginx负载均衡调度算法**

| 调度算法   | 概述                                                         |
| ---------- | ------------------------------------------------------------ |
| 轮询       | 按时间顺序逐一分配到不同的后端服务器(默认)                   |
| weight     | 加权轮询,weight值越大,分配到的访问几率越高                   |
| ip_hash    | 每个请求按访问IP的hash结果分配,这样来自同一IP的固定访问一个后端服务器 |
| url_hash   | 按照访问URL的hash结果来分配请求,是每个URL定向到同一个后端服务器 |
| least_conn | 最少链接数,那个机器链接数少就分发                            |

- 1.Nginx负载均衡[wrr]轮询具体配置

~~~nginx
upstream load_pass {
    server 10.0.0.7:80;
    server 10.0.0.8:80;
}
~~~

- 2.Nginx负载均衡[weight]权重轮询具体配置

~~~~nginx
upstream load_pass {
    server 10.0.0.7:80 weight=5;
    server 10.0.0.8:80;
}
~~~~

- 3.Nginx负载均衡ip_hash具体配置, 不能和weight一起使用。


  如果客户端都走相同代理, 会导致某一台服务器连接过多

~~~nginx
upstream load_pass {
    ip_hash;
    server 10.0.0.7:80 weight=5;
    server 10.0.0.8:80;
}
~~~

**7.Nginx负载均衡后端状态**

​	后端Web服务器在前端Nginx负载均衡调度中的状态

| 状态         | 概述                              |
| ------------ | --------------------------------- |
| down         | 当前的server暂时不参与负载均衡    |
| backup       | 预留的备份服务器                  |
| max_fails    | 允许请求失败的次数                |
| fail_timeout | 经过max_fails失败后, 服务暂停时间 |
| max_conns    | 限制最大的接收连接数              |

>
> 企业案例:使用nginx负载均衡时，如何将后端请求超时的服务器流量平滑的切换到另一台上。如果后台服务连接超时，Nginx是本身是有机制的，如果出现一个节点down掉的时候，Nginx会更据你具体负载均衡的设置，将请求转移到其他的节点上，但是，如果后台服务连接没有down掉，但是返回错误异常码了如:504、502、500，应该如何处理。
> 可以在负载均衡添加如下配置proxy_next_upstream http_500 | http_502 | http_503 | http_504 |http_404;意思是，当其中一台返回错误码404,500...等错误时，可以分配到下一台服务器程序继续处理，提高平台访问成功率。
>
> ​	nginx本身是有剔除机制,  指的是 后端的nginx没有正常工作
> ​	proxy_next_upstream nginx是正常工作,只不过后端的php或者其他程序出现问题  502
>
> ```nginx
> server {
>     listen 80;
>     server_name xuliangwei.com;
>     location / {
>     proxy_pass http://node;
>     proxy_next_upstream error timeout http_500 http_502 http_503 http_504;
> }
> }
> ```
> 

**8.七层负载均衡实现Redis会话共享?**

在使用负载均衡的时候会遇到会话保持的问题，可通过如下方式进行解决

- 1.使用nginx的ip_hash，根据客户端的来源IP，将请求分配到相同服务器上
- 2.基于服务端的Session会话共享（mysql/memcache/redis/file）

在解决负载均衡会话问题我们需要了解session和cookie。

- 1.用户第一次请求服务端网站时，服务端会生成对应的session_id，然后存储至客户端浏览器的cookie中。
- 2.客户端尝试登陆服务端网站时，浏览器的请求头自动携带cookie信息，在cookie信息中保存的则是session_id。
- 3.客户端登陆服务端网站后，服务端会将session_id存储在本地文件中, 当用户下次请求网站时会去查询用户提交的cookie作为key去存储里找对应的value(session)

注意: 同一域名下的网站登陆后cookie都是一样的。所以无论负载后端有几台服务器,无论请求分配到哪一台服务器上同一用户的cookie是不会发生变化的。也就是说cookie对应的session也是唯一的。所以，这里只要保证多台业务服务器访问同一个共享服务器(memcache/redis/mysql/file)就行了。

> ***实现***
>
> 1、粘性session
> 粘性session是指Ngnix每次都将同一用户的所有请求转发至同一台服务器上，及Nginx的 IP_hash。
>
> 2、session复制
> 即每次session发生变化时，创建或者修改，就广播给集群中的服务器，使所有的服务器上的session相同。
>
> 3、session持久化 ( 慢 )
> 将session存储至数据库中，像操作数据一样操作session。
>
> 4、session共享
> 缓存session至内存数据库中，使用redis ( 内存-->刷到磁盘  )，memcached (内存数据库)。

### 示例

- 1.在 172.16.1.8  和  172.16.1.7 安装 phpmyadmin     
  	分别进行测试-->测试登录

```bash
#1.安装phpmyadmin（web01和web02上都装）
[root@web01 conf.d]# cd /code
[root@web01 code]# wget https://files.phpmyadmin.net/phpMyAdmin/4.8.4/phpMyAdmin-4.8.4-all-languages.zip
[root@web01 code]# unzip phpMyAdmin-4.8.4-all-languages.zip

#2.配置phpmyadmin连接远程的数据库
[root@web01 code]# cd phpMyAdmin-4.8.4-all-languages/
[root@web01 phpMyAdmin-4.8.4-all-languages]# cp config.sample.inc.php config.inc.php
[root@web01 phpMyAdmin-4.8.4-all-languages]# vim config.inc.php
/* Server parameters */
$cfg['Servers'][$i]['host'] = '172.16.1.51';
```
- 2.接入负载均衡  ---> 代理至后端2台主机  

```bash
[root@lb01 conf.d]# cat proxy_php.oldxu.com.conf 
upstream  php {
	server 172.16.1.7;
	server 172.16.1.8;
}

server {
	listen 80;
	server_name php.oldxu.com;
	location / {
		proxy_pass http://php;
		proxy_set_header Host $http_host;
	}
}
```

- 3.发现无法正常登陆
  1.解决方法: 

  在负载均衡上配置  ip_hash 会话保持   (  造成用户仅访问后端的某一台主机  )

```bash
[root@lb01 conf.d]# cat proxy_php.oldxu.com.conf 
upstream  php {
	ip_hash;
	server 172.16.1.7;
	server 172.16.1.8;
}

server {
	listen 80;
	server_name php.oldxu.com;
	location / {
		proxy_pass http://php;
		proxy_set_header Host $http_host;
	}
}
```

- 4.既希望能够实现流量的均摊,又希望会话的问题得以保持, 所以引入了redis

```bash
#1.安装redis
	[root@db01 ~]# yum install redis -y
#2.配置redis
	[root@db01 ~]# sed -i '/^bind/c bind 127.0.0.1 172.16.1.51' /etc/redis.conf
#3.启动redis
	[root@db01 ~]# systemctl enable redis
	[root@db01 ~]# systemctl start redis

#4.改造php, session写本地修改为写入redis中  (所有的web上都需要配置
	#前提:  已经安装过了redis的模块---> php71w-pecl-redis
	##1.修改php存储session至redis中
	[root@web01 ~]# vim /etc/php.ini
	session.save_handler = redis
	session.save_path = "tcp://172.16.1.51:6379?weight=1"

	##2.修改php-fpm 注释默认存储session的位置
	[root@web01 ~]# vim /etc/php-fpm.d/www.conf
	;php_value[session.save_handler] = files
	;php_value[session.save_path]  =  /var/lib/php/session

	##3.将修改后的配置文件,推送至172.16.1.8
	[root@web01 ~]# scp /etc/php.ini root@172.16.1.8:/etc/  
	[root@web01 ~]# scp /etc/php-fpm.d/www.conf  root@172.16.1.8:/etc/php-fpm.d/www.conf

	##4.重启172.16.1.7 172.16.1.8两台服务器的php-fpm
	[root@web02 conf.d]# systemctl restart php-fpm
#5.测试效果
	##1.浏览器登录测试  (ok)
	##2.查看redis的sessionID和    浏览器cookie中提交的sessionID是否一致
	[root@db01 ~]# redis-cli 
	127.0.0.1:6379> keys *
	1) "PHPREDIS_SESSION:38ecc8696c70a7252d943e7cb9b20f70"
```








	

