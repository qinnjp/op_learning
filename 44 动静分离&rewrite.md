## 动静分离



**1.什么是动静分离?**

​	动静分离, 通过中间件将动静分离和静态请求进行分离。
​	那为什么要通过中间件将动态请求和静态请求进行分离? 减少不必要的请求消耗, 同时能减少请求的延时。
​	通过中间件将动态请求和静态请求分离，逻辑图如下

![img](https://img2018.cnblogs.com/blog/1488697/201906/1488697-20190603222140533-290568723.png)

​	动静分离只有好处: 动静分离后, 即使动态服务不可用, 但静态资源不会受到影响

**2.为什么要做动静分离?**

​	静态由Nginx处理, 动态由PHP处理或Tomcat处理....
​	因为Tomcat程序本身是用来处理jsp代码的,但tomcat也能处理静态资源.
​	tomcat本身处理静态效率不高,还会带来资源开销.

**3.如何实现动静分离?**

​	Nginx根据客户端请求的url来判断请求的是否是静态资源，如果请求的url包含jpg、png，则由Nginx处理。
如果请求的url是.php或者.jsp等等，这个时候这个请求是动态的，将转发给tomcat处理。

​	总结来说，Nginx是通过url来区分请求的类型，并转发给不同的服务

![1569324755398](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1569324755398.png)

**4.单机实现动静分离实战**

~~~bash
[root@web01 ~]# yum install java tomcat -y
[root@web01 ~]# mkdir /usr/share/tomcat/webapps/ROOT		#主要站点根目录
[root@web01 ~]# vi /usr/share/tomcat/webapps/ROOT/index.jsp
<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>
<html>
  <head>
	<title>Nginx+Tomcat</title>
  </head>
  <body>
	  <%
		Random rand = new Random();
		out.println("<h2>动态资源</h2>");
		out.println(rand.nextInt(99)+100);
	%>
	<h2>静态图片</h2>
	<img src="nginx.png" />
  </body>
</html>

[root@web01 ~]# wget -O /usr/share/tomcat/webapps/ROOT/nginx.png http://nginx.org/nginx.png
[root@web01 ~]# systemctl start tomcat

  # tomcat监听在8080端口上:
  
 #配置Nginx 
[root@web01 conf.d]# cat ds.oldxu.com.conf 
server {
	listen 80;
	server_name ds.oldxu.com;
	location / {
	proxy_pass http://127.0.0.1:8080;
}
location ~* \.(png|gif|jpg|mp4)$ {
	root /images;
	expires 1d;
}
}
~~~

**5.集群实现动静分离实战**

![1569325124169](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1569325124169.png)

~~~bash
#负载均衡
[root@lb01 conf.d]# cat  proxy_ds.oldxu.com.conf

upstream java {
	server 172.16.1.7:8080;
} 
upstream static {
	server 172.16.1.8:80;
}
server {
	listen 80;
	server_name ds.oldxu.com;
	location / {
		proxy_pass http://java;
		include proxy_params;
	}

	location ~* \.(png|gif|jpeg)$ {
		proxy_pass http://static;
		expires 2d;
		include proxy_params;
	}
}
#动态Tomcat web01
[root@web01 images]# cat /etc/nginx/conf.d/ds.oldxu.com.conf
server {
	listen 80;
	server_name ds.oldxu.com;

	location / {
		proxy_pass http://127.0.0.1:8080;
	}

}

#静态图片 web02
[root@web02 images]# cat /etc/nginx/conf.d/ds.oldxu.com.conf 
server {
	listen 80;
	server_name ds.oldxu.com;
	location ~* \.(png|gif|jpg)$ {
		root /images;
	}
}

~~~

## rewrite

**1.什么是Rewrite?**

​	Rewrite主要实现url地址重写, 以及地址重定向，就是将用户请求web服务器的地址重新定向到其他URL的过程。

**2.Rewrite使用场景?**

- 1.地址跳转，用户访问www.xuliangwei.com/class这个URL时，将其定向至一个新的域名class.xuliangwei.com
- 2.协议跳转，用户通过http协议请求网站时，将其重新跳转至https协议方式
- 3.伪静态，将动态页面显示为静态页面方式的一种技术, 便于搜索引擎的录入, 同时减少动态URL地址对外暴露过多的参数, 提升更高的安全性。
- 4.搜索引擎，SEO优化依赖于url路径, 好记的url便于支持搜索引擎录入

**3.Rewrite实现原理?**

```bash
#rewrite表达式可以应用在server,location, if标签下
Syntax: rewrite regex replacement [flag];
Default: --
Context: server, location, if

#用于切换维护页面场景
#rewrite ^(.*)$ /page/wh.html break;
```

![img](https://images2018.cnblogs.com/blog/1459159/201808/1459159-20180806210351227-896085145.png)

**4.Rewrite URL重写配置场景**?

- 1.set自定义变量
  - set指定语法

~~~nginx
Syntax: set $variable value
Default:-
Context:server,location,if
~~~

- set指令场景示例

  需求：将用户请求url.oldxu.com.cn重定向至url.oldxu.com/zh

~~~bash
[root@web01 conf.d]# cat url.oldxu.com.conf 
server {
    listen 80;
    server_name url.oldxu.com.cn;

	set $language zh;
	rewrite ^/$ http://url.oldxu.com/$language/;
}
server {
	listen 80;
	server_name url.oldxu.com;
	location / {
		root /data;
	}
}

~~~

- 2.if判断语句

  1.if指令语法

~~~nginx
Syntax: if (condition) {...}
Default:-
Context:server,location

# ~ 模糊匹配
# ~* 不区分大小写的匹配
# ！~ 不匹配
# = 精确匹配
~~~

  	2.if指令场景示例：

​		需求1：将用户请求url.oldxu.com.cn跳转至url.oldxu.com/zh

​		需求2：将用户请求url.oldxu.com.jp跳转至url.oldxu.com/jp

~~~bash
[root@web01 conf.d]# cat url.oldxu.com.conf 
server {
    listen 80;
    server_name url.oldxu.com.cn url.oldxu.com.jp;

	#判断
	if ($http_host ~* cn) {
		set $language zh;	
	}
	if ($http_host ~* jp) {
		set $language jp;	
	}	
	rewrite ^/$ http://url.oldxu.com/$language/;
}
server {
	listen 80;
	server_name url.oldxu.com;
	location / {
		root /data;
	}
}

~~~

​	3.需求：根据用户浏览器的使用的语言，自动判断并跳转到不同的语言界面。

~~~bash
[root@web01 conf.d]# cat url.oldxu.com.conf 
server {
	listen 80;
	server_name url.oldxu.com;

	location / {
		if ($http_accept_language ~* "en") {
			set $language en;	
		}
		if ($http_accept_language ~* "zh") {
			set $language zh;	
		}
	
		root /data/$language;
	}
}

~~~

​	4.需求nginx过滤请求中包含a1=3256的http请求返回ok（可以调至某台联通服务器的特定的端口做处理）

~~~nginx
if ($request_uri ~*  "a1=3256") {
			return 200 "ok";
}
if ($request_uri ~*  "a1=3256") {
			proxy_pass  http://10.0.0.7:8080;
}
~~~

- 3.return 返回数据

  1.return指令语法

~~~nginx
Syntax: return code [text];
return code url;
return URL;
Default:-
Context:server,location,if

#状态码
#状态码 字符串
#状态码 URL    301 302
~~~

~~~nginx
default_type text/html;
if ($request_uri ~* "a1=3526") {
	return 200 "ok"; #跳转到界面
}
if ($request_uri ~* "git"){
	return 403;              #返回403
}
if ($request_uri ~* "^/test") {
	return 302 "https://www.jd.com";  #跳转到京东
}
~~~

- break

~~~nginx
server {
    listen 80;
    server_name url.oldxu.com;
    root /code;

    location / {
        rewrite /1.html /2.html ；break;
        rewrite /2.html /3.html;
    }

    location /2.html {
        rewrite /2.html /a.html;
    }

    location /3.html {
        rewrite /3.html /b.html;
    }
} 
[root@web01]# echo "1.html" >/code/1.html
[root@web01]# echo "2.html" >/code/2.html
[root@web01]# echo "3.html" >/code/3.html
[root@web01]# echo "a.html" >/code/a.html
[root@web01]# echo "b.html" >/code/b.html
#测试结果：当请求/1.html,最终会访问/2.html
#因为：在location{}内部，遇到break，在本location{}内以及后面的所有location{}内的所有指令都将不在执行。
~~~

- rewrite

  | #       | 关键字  | 正则  | 替代内容    | flag标记 |
  | ------- | ------- | ----- | ----------- | -------- |
  | Syntax: | rewrite | regex | replacement | [flag];  |

  

跳转  :
重定向:

> #flag
> last       		#本条规则匹配完成后，继续向下匹配新的location URI规则 (开发| 伪静态)
>
> break      		#本条规则匹配完成即终止，不再匹配后面的任何规则(挂维护页)
> redirect   		#返回302临时重定向, 地址栏会显示跳转后的地址
> permanent  		#返回301永久重定向, 地址栏会显示跳转后的地址

- 根据用户浏览器的使用的语言，自动判断并跳转到不同的语言界面。

~~~nginx
server {
    listen 80;
    server_name url.oldxu.com;
    root /data;

	set $language /default;
	if ( $http_accept_language ~* zh ) {
		set $language /zh;
	}
	if ( $http_accept_language ~* en ) {
		set $language /en;
	}
	if ( $http_accept_language ~* ja ) {
		set $language /jp;
	}

	rewrite ^/$ $language;
	
	location / {
		index index.html;
	}
}
~~~

- 维护

~~~nginx
#永久维护
server {
    listen 80;
    server_name url.oldxu.com;
    root /data;

	rewrite ^(.*)$ /wh.png break;
}
#临时维护(jd)
#error_page 403 404 500 502 /wh.png;
    #error_page 403 404 500 502 http://$http_host;

    error_page 403 404 500 502 @temperror;
    location @temperror {
            rewrite ^(.*)$ http://$http_host;
    }
~~~

- 需求: 用户通过手机设备访问url.oldxu.com，跳转至url.oldxu.com/m

~~~nginx
server {
    listen 80;
    server_name url.oldxu.com;
    root /data;

    if ($http_user_agent ~* "android|iphone|ipad") {
            rewrite ^/$ /m;
    }
}
~~~

- 需求: 用户通过手机设备访问url.oldxu.com，跳转至m.oldxu.com

~~~nginx
server {
    listen 80;
    server_name url.oldxu.com;
    root /data;

	if ($http_user_agent ~* "android|iphone|ipad") {
		rewrite ^/$  http://m.oldxu.com;
	}
}
server {
	listen 80;
	server_name m.oldxu.com;
	root /data/m;
	
	location / {
		index index.html;
	}
}
~~~

- 需求: 用户访问oldxu.com/test，跳转至https://xuliangwei.com

~~~nginx
server {
	listen 80;
	server_name url.oldxu.com;
	root /data;

	if ($request_uri ~* "^/test") {
		#rewrite ^(.*)$ https://www.xuliangwei.com/;
		return 302 https://www.xuliangwei.com/;
	}
	location / {
		index index.html;
	}
}

~~~

- 需求: 用户访问course-11-22-33.html实际上真实访问是/course/11/22/33/course_33.html

~~~
[root@web01 conf.d]# cat url.oldxu.com.conf 
server {
    listen 80;
    server_name url.oldxu.com;
	root /data;
	location /  {
		index index.html;
				#用户访问的url		#文件真实位置
		rewrite ^/(.*)-(.*)-(.*)-(.*).html /$1/$2/$3/$4/$1_$4.html;
	}
}
~~~

