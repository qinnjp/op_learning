***1.nginx+tomcat集群架构介绍***

- tomcat集群能带来什么
1.提高服务的性能，并发能力，以及高可用性。
2.提高项目架构的扩展能力

- tomcat集群实现原理
通过nginx负载均衡进行请求转发
- tomcat集群架构
![image.png](https://upload-images.jianshu.io/upload_images/18911567-33b2c4401d44dbce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
***2.nginx+tomcat集群架构实战***
配置负载均衡
~~~nginx
[root@lb01 conf.d]# cat proxy_zrlog.oldxu.com.conf 
upstream  zrlog {
	server 172.16.1.7:8080;
	server 172.16.1.8:8080;
}

server {
	listen 80;
	server_name zrlog.oldxu.com;

	location / {
		proxy_pass http://zrlog;
		include proxy_params;
	}
}
~~~
***3.nginx+tomcat集群回话共享 redis  cluster***
session测试代码用例
- 1.配置虚拟主机
~~~java
[root@web01 conf]# vim /soft/tomcat/conf/server.xml 
<!--session站点-->
  <Host name="session.oldxu.com"  appBase="/code/session"
        unpackWARs="true" autoDeploy="true">
  </Host>
~~~
- 2.准备index.jsp文件(为了区分需要调整输出的web01 web02)

```java
[root@web01 conf]# cat /code/session/ROOT/index.jsp 
<body>
<%
//HttpSession session = request.getSession(true);
System.out.println(session.getCreationTime());
out.println("<br> web01 SESSION ID:" + session.getId() + "<br>");
out.println("Session created time is :" + session.getCreationTime()
+ "<br>");
%>
</body>

```

- 3.下载TomcatClusterRedisSessionManager (所有web集群都需要操作)
~~~bash
#GitHub地址    https://github.com/ran-jit/tomcat-cluster-redis-session-manager

[root@tomcat ~]# wget https://github.com/ran-jit/tomcat-cluster-redis-session-manager/releases/download/3.0.3/tomcat-cluster-redis-session-manager.zip
[root@tomcat ~]# unzip tomcat-cluster-redis-session-manager.zip
[root@web01 ~]# cd tomcat-cluster-redis-session-manager
~~~

1.拷贝jar包
~~~bash
[root@web01 tomcat-cluster-redis-session-manager]# cp lib/* /soft/tomcat/lib/
~~~
2.拷贝tomcat连接redis配置文件
~~~~bash
[root@web01 tomcat-cluster-redis-session-manager]# cp conf/redis-data-cache.properties /soft/tomcat/conf/
~~~
3.修改redis-data-cache.properties
~~~java
[root@web01 ~]# vim /soft/tomcat/conf/redis-data-cache.properties
...
redis.hosts=172.16.1.51:6379
redis.password=123456			#有密码就写密码,没有不要写
...
~~~

4.添加如下两行至tomcat/conf/context.xml
~~~java
[root@web01 ~]# vim /soft/tomcat/conf/context.xml
<Context>
	.....
	<Valve className="tomcat.request.session.redis.SessionHandlerValve" />
	<Manager className="tomcat.request.session.redis.SessionManager" />
	....
</Context>
~~~~

5.修改tomcat/conf/web.xml 配置文件session的超时时间 ,单位是分钟
```bash
	<session-config>
			<session-timeout>60</session-timeout>		#根据情况调整
	</session-config>
```

6.安装redis，当然也可以自行搭建redis集群，anyway
```bash
[root@redis ~]# yum install redis -y
[root@redis ~]# cat /etc/redis.conf
...
bind 172.16.1.51 127.0.0.1
requirepass 123456				#如果不需要密码,则不要配置
...
[root@redis ~]# systemctl start redis
[root@redis ~]# systemctl enable redis
```

7.重启多台机器的Tomcat

8.接入负载均衡,通过负载均衡轮询调度检查是否正常


9.如果session会话不正常:
	将域名解析到指定的服务器,通过8080的方式去访问,测试,检查日志.
	
***4.nignx+tomcat集群全站https***
单台：
        1.http接收器修改为80端口  ---> 443
        2.配置443的证书
集群：
```nginx
[root@lb01 conf.d]# cat proxy_zrlog.oldxu.com.conf 
upstream  zrlog {
	server 172.16.1.7:8080;
	server 172.16.1.8:8080;
}

server {
	listen 443 ssl;
	ssl_certificate ssl_key/server.crt;
	ssl_certificate_key ssl_key/server.key;
	server_name zrlog.oldxu.com;

	location / {
		proxy_pass http://zrlog;
		include proxy_params;
	}
}
server {
	listen 80;
	server_name zrlog.oldxu.com;
	return 302 https://$http_host$request_uri;
}

```