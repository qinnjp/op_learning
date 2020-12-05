# LNMP架构

**1.什么是LNMP**

​	LNMP是一套技术的组合，L=linux，N=nginx，M~=MySQL，P~=PHP

**2.LNMP架构是如何工作的**

​	首先Nginx服务是不能处理动态请求的，那么当用户发起动态请求时，Nginx是又如何进行处理的。当用户发起http请求，请求会被Nginx处理，如果是静态资源请求Nginx则直接返回，如果是动态请求Nginx则通过fastcgi协议转交给后端的PHP程序处理，具体如下图所示。

![1568799683747](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1568799683747.png)

~~~bash
location / {
	index index.php;
}

location ~ \.php$ {
	fastcgi_pass 127.0.0.1:9000;
}

location ~ \.(jpg|png|gif)$ {
	root /code/images;
}
~~~



**3.Nginx与PHP，MySQL之间是如何工作的**

- 用户通过http协议发起请求，请求会先抵达LNMP架构中的Nginx
- Nginx会根据用户的请求进行Location规则匹配
- Location如果匹配到请求是静态，则由Nginx读取本地直接返回
- location如果匹配到请求时动态，则由nginx将请求转发给fastcgi协议
- fastcgi收到后会将请求交给php-fpm管理进程接受到后会调用具体的工作进程warrap
- warrap进程会调用php程序进行解析，如果只是解析代码，php直接返回
- 如果有查询数据库操作，则由PHP连接数据库（用户 密码 ip）发起查询的操作

![1568801491064](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1568801491064.png)

**4.如何安装LNMP架构**

- 安装（若有老版本PHP先卸载老版本PHP）

  ~~~bash
  [root@web01 ~]# cat /etc/yum.repos.d/php.repo 
  [webtatic-php]
  name = php Repository
  baseurl = http://us-east.repo.webtatic.com/yum/el7/x86_64/
  gpgcheck = 0
  
  #安装nginx
  yum install nginx -y 
  #安装PHP
  yum remove php-mysql-5.4 php php-fpm php-common
  yum -y install php71w php71w-cli php71w-common php71w-devel php71w-embedded php71w-gd php71w-mcrypt php71w-mbstring php71w-pdo php71w-xml php71w-fpm php71w-mysqlnd php71w-opcache php71w-pecl-memcached php71w-pecl-redis php71w-pecl-mongodb
  #安装数据库
  yum install mariadb mariadb-server -y
  ~~~

- 启动nginx php-fpm

  ~~~bash
  systemctl start nginx
  systemctl start php-fpm
  ~~~

**5.Fastcgi代理配置语法**

- 设置fastcgi服务器的地址，该地址可以指定为域名或ip地址，以及端口

~~~bash
#语法示例
fastcgi_pass localhost:9000;
~~~

- 设置fastcgi默认的首页文件，需要结合fastcgi_param一起设置

- 通过fastcgi_param设置变量，并将设置的变量传递到后端的fastcgi服务

~~~bash
#语法示例

fastcgi_index index.php;
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
~~~

![1568807932912](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1568807932912.png)

**6.Nginx与PHP集成的原理。**

```bash
#编写能解析PHP的Nginx配置文件
[root@web01 conf.d]# cat php.oldxu.com.conf 
server {
	listen 80;
	server_name php.oldxu.com;
	root /code;

	location / {
		index index.php;
	} 

	location ~ \.php$ {
		fastcgi_pass 127.0.0.1:9000;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
}

#2.编写PHP代码,测试访问效果.
[root@web01 conf.d]# cat /code/info.php
<?php
	phpinfo();
?>

#3.host劫持
```
**7.PHP与MySQL集成的原理。**

```bash
#1.启动数据库
[root@web01 ~]# systemctl start mariadb

#2.配置连接密码
[root@web01 ~]# mysqladmin password oldxu.com

#3.测试登录mysql
[root@web01 ~]# mysql -uroot -poldxu.com
MariaDB [(none)]>

#4.编写php连接数据库的代码
[root@web01 ~]# /code/mysqli.php
<?php
    $servername = "localhost";
    $username = "root";
    $password = "oldxu.com";

    // 创建连接
    $conn = mysqli_connect($servername, $username, $password);

    // 检测连接
    if (!$conn) {
        die("Connection failed: " . mysqli_connect_error());
    }
    echo "php连接MySQL数据库成功";
?>
#5.可以直接使用php命令测试
	[root@web01 ~]# php /code/mysqli.php

#6.也可以通过浏览器的方式去测试
```

**8.通过LNMP架构部署Wordpress、Wecenter、edusoho、phpmyadmin、ecshop、**

- 1.编写Nginx集成PHP的配置文件  (定义域名以及站点的目录位置)

	[root@web01 conf.d]# cat blog.oldxu.com.conf 
	server {
		listen 80;
		server_name blog.oldxu.com;
		root /code/wordpress;
	location / {
		index index.php;
	}
	
	location ~ \.php$ {
		fastcgi_pass 127.0.0.1:9000;
		fastcgi_param   SCRIPT_FILENAME  $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
	}
- 2.根据Nginx配置,初始化环境,然后上传代码

```bash
#1.准备站点目录
[root@web01 conf.d]# mkdir /code
#2.下载wordpress代码
[root@web01 conf.d]# cd /code
[root@web01 code]# tar xf wordpress-5.2.3-zh_CN.tar.gz
#3.创建数据库名
[root@web01 code]# mysql -uroot -poldxu.com

MariaDB [(none)]> create database wordpress;
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
| wordpress          |
+--------------------+
5 rows in set (0.01 sec)
#4.统一Nginx  PHP的权限  为  www
[root@web01 code]# groupadd www -g 666
[root@web01 code]# useradd -u666 -g666 www

[root@web01 code]# sed -i '/^user/c user www;' /etc/nginx/nginx.conf
[root@web01 code]# chown -R www.www /code
[root@web01 code]# systemctl restart nginx

[root@web01 code]# sed -i '/^user/c user = www' /etc/php-fpm.d/www.conf 
[root@web01 code]#  sed -i '/^group/c group = www' /etc/php-fpm.d/www.conf
[root@web01 code]# systemctl restart php-fpm
```


	


















