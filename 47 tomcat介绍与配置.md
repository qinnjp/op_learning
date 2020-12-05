***1.JVM基本介绍***
- jvm概述
java业务都是运行在虚拟机上的，Java虚拟机简称为jvm
- 为什么java需要jvm虚拟机
 Java ---> 编译型语言
 PHP ---> 解释型语言
C语言---> 编译型语言

早期的C语言不支持跨平台编译在不同的平台上运行需要进行分别编译。
而java不同，java可以跨平台，只需要将源代码进行一次编译，能够再多处运行，但需要在各个平台上运行一个jvm，这样我们能将java编译好的war、包在Windows和linux平台上运行起来，无需我们重复编译。
- java环境jre、和jdk区别在哪里？
  环境:
  - jre:   java运行环境  包含 jvm
  - jdk:   java开发环境  包含 jre
  		

  只想运行java代码,只需要jre即可
  既想要运行环境,开发环境   jdk
  ***tomcat和nginx的区别？
- 什么是tomcat？
tomcat和此前学习过的linux类似，也是一个web的服务型器软件。
只不过tomcat是基于java开发的web容器，主要用于发布java代码，网页代码jsp。
- tomcat与nignx的区别
nginx仅能支持静态资源解析，而tomcat则支持java开发的jsp动态资源。
nginx适合做前端的负载均衡，而tomcat则支持做后端应用服务处理。
通常情况下，企业会使用nginx+tomcat结合，有nginx处理静态资源，tomcat出来动态资源。
***3.tomcat安装配置启动***
- yum安装
~~~
[root@web01 ~]# yum install java -y
~~~
   二进制安装tomcat
~~~
[root@web01 ~]# mkdir /soft && cd /soft
[root@web01 soft]# wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.26/bin/apache-tomcat-9.0.26.tar.gz
[root@web01 soft]# tar xf apache-tomcat-9.0.26.tar.gz 
[root@web01 soft]# ln -s /soft/apache-tomcat-9.0.26 /soft/tomcat
[root@web01 soft]# /soft/tomcat/bin/startup.sh
[root@web01 soft]# netstat -lntp|grep java
tcp6       0      0 :::8009            :::*        LISTEN      8500/java           
tcp6       0      0 :::8080            :::*        LISTEN      8500/java           
tcp6       0      0 127.0.0.1:8005     :::*        LISTEN      8500/java    
~~~
- 二进制:
jdk使用二进制
tomcat使用二进制
~~~
[root@es-node1 ~]# mkdir /soft/
[root@es-node1 ~]# tar xf jdk-8u60-linux-x64.tar.gz -C /app/
[root@es-node1 ~]# ln -s /soft/jdk1.8.0_60 /soft/jdk
[root@es-node1 ~]# vim /etc/profile
#...最后面添加...
export JAVA_HOME=/app/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar

[root@es-node1 ~]# tar xf apache-tomcat-9.0.26.tar.gz -C /soft
[root@es-node1 ~]# /soft/apache-tomcat-9.0.26/bin/startup.sh 
~~~
***4.tomcat配置文件详解***
~~~
[root@web01 tomcat]# ll
total 124
drwxr-x--- 2 root root  4096 Sep 27 09:38 bin  #里面包含启动和关闭的脚本，以及依赖的一些jar文件
drwx------ 3 root root   275 Sep 27 15:45 conf  #tomcat配置文件目录
drwxr-x--- 2 root root  4096 Sep 27 09:38 lib  #运行需要加载的jar包
drwxr-x--- 2 root root   277 Sep 27 11:56 logs  #在运行过程中产生的日志文件
drwxr-x--- 2 root root    30 Sep 27 17:10 temp  #存放临时文件
drwxr-x--- 7 root root    81 Sep 16 23:52 webapps  #默认站点目录
drwxr-x--- 3 root root    22 Sep 27 09:42 work  #运行时产生的缓存文件
~~~
>一个server表示一个tomcat实例
>一个server中包含多个Connector连接器，Connector的主要功能是接受、响应用户请求。
>service的作用是：将connector关联至engine(catalina引擎)
>一个host就是一个站点，类似于nginx的多站点
>context类似于nginx中location的概念
>
>![image.png](https://upload-images.jianshu.io/upload_images/18911567-51ccf090d8b486e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>***5.Tomcat配置虚拟主机		---> 一台服务器运行多个站点***
~~~
<!--tomcat虚拟主机-->
<Host name="tomcat1.oldxu.com"  appBase="/code1"
	unpackWARs="true" autoDeploy="true">

<!--类似于nginx的location  path是访问的路径 ->映射 docBase是真实的路径-->
<Context docBase="/code1/admin" path="/test" reloadable="true"/>

<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
		prefix="tomcat1_access_log" suffix=".txt"
		pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
~~~
无法启动:
	1.替换配置文件:   pkill java  然后在启动
	2.配置文件写错   
	3.虚拟主机是添加,不要修改
	4.context如果写了,一定要有对应的目录,不然整体就报错
	/soft/tomcat/logs/catalina.out
***6.Tomcat部署博客项目zrlog***

域名: zrlog.oldxu.com:8080
站点目录:  /code/zrlog      <--ROOT
	
10.0.0.7  tomcat   端口 8080		10.0.0.51 mysql    端口 3306
- 1.配置server.xml文件 ,新增在 engline
~~~
[root@web01 ~]# vim /soft/tomcat/conf/server.xml
 <!--zrlog站点-->
<Host name="zrlog.oldxu.com"  appBase="/code/zrlog"
      unpackWARs="true" autoDeploy="true">

  <Va0lve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
         prefix="zrlog_access_log" suffix=".txt"
         pattern="%h %l %u %t &quot;%r&quot; %s %b" />
~~~
- 2.创建站点目录,上传zrlog的war包
~~~
[root@web01 ~]# mkdir /code/zrlog
[root@web01 ~]# cd /code/zrlog/
[root@web01 zrlog]# rz ROOT.war    <---这个是zrlog的war包,名称就叫ROOT
~~~
- 3.重启Tomcat服务
~~~
[root@web01 zrlog]# /soft/tomcat/bin/shutdown.sh && /soft/tomcat/bin/startup.sh && tail -f /soft/tomcat/logs/catalina.out
~~~
- 4.配置域名劫持


- 5.在172.16.1.51的数据库上,创建一个zrlog的库,配置授权访问用户
~~~
[root@db01 ~]# mysql -uroot -poldxu.com
MariaDB [(none)]> create database zrlog charset utf8;
	<---此前配置过all用户,可以复用
MariaDB [(none)]> grant all privileges on *.* to 'all'@'%' identified by 'oldxu.com';
~~~

源码包-->jar包--war包的关系?
	
源码包  -->  由开发人员编写的
	
maven
	
jar     -->  源码包编译

无法独立运行, 需要被某个程序所依赖    mysql连接
可以独立运行, java -jar xx.jar 启动
https://gitee.com/chejiangyi/dingding-sonar

war		-->  源码包编译, 可以直接放在tomcat中进行部署  (这种类型居多)

源码-->maven编译-->jar或者war包
war包直接放入tomcat即可运行, war在运行过程中需要依赖 jar包
jar包 分为两种,   可独立运行(对外提供服务),    不可独立运行(被war依赖)

***6.tomcat配置基础认证***
如何开启 Server Status  Host Manager页面
~~~
# 1.配置conf/tomcat-users.xml
	<role rolename="manager-gui"/>
	<user username="tomcat" password="123456" roles="manager-gui"/>


# 2.如果访问还是403，是因为tomcat默认仅运行本地访问该管理页面，需要允许同网段主机访问
	[root@web01 ~]# ll /soft/tomcat/webapps/manager/
	[root@web01 ~]# ll /soft/tomcat/webapps/host-manager/
		
	[root@es-node1 tomcat]# vim 项目目录下/META-INF/context.xml
	allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
	#修改为
		allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|10\.0\.0\.\d+" />
~~~
***7.启用zrlog的基础认证 ---> zrlog.oldxu.com/admin***
~~~
[root@es-node1 tomcat]# vim /code/zrlog/ROOT/WEB-INF/web.xml
<web-app>
...
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>zrlog</web-resource-name>
            <url-pattern>/admin/*</url-pattern>
        </web-resource-collection>
    
        <auth-constraint>
            <role-name>zrlog_role</role-name>
        </auth-constraint>
    </security-constraint>
    
    <login-config>
        <auth-method>BASIC</auth-method>
        <realm-name>Default</realm-name>
    </login-config>
...
</web-app>


#配置用户名密码，关联对应的角色(多个role不要使用相同用户)
[root@es-node1 tomcat]# vim /soft/tomcat/conf/tomcat-users.xml
<role rolename="zrlog_role"/>
<user username="tomcat" password="123456" roles="zrlog_role"/>

#重启tomcat
[root@es-node1 ~]# /soft/tomcat/bin/shutdown.sh && /soft/tomcat/bin/startup.sh  
~~~
***8.使用Nginx替代tomcat基础认证***
~~~
server {
listen 80;
server_name zrlog.oldxu.com;

location / {
proxy_pass http://127.0.0.1:8080;
proxy_set_header Host $http_host;
}

location /admin {
auth_basic           "closed site";
auth_basic_user_file conf/auth_conf;
proxy_pass http://127.0.0.1:8080;
proxy_set_header Host $http_host;
}
}
~~~
***9.部署多节点Tomcat-->mysql***
10.0.0.7   ---> 10.0.0.51
10.0.0.8   ---> 10.0.0.51


1.安装jdk
~~~
[root@web02 ~]# yum install java -y
~~~
2.安装tomcat  部署代码  (scp)
在web01上操作
~~~	
[root@web01 ~]# scp -rp /soft root@172.16.1.8:/
[root@web01 ~]# scp -rp /code/zrlog root@172.16.1.8:/code/
~~~
在web02上操作
~~~
[root@web02 soft]# rm -rf /soft/tomcat/
[root@web02 soft]# ln -s /soft/apache-tomcat-9.0.26 /soft/tomcat
[root@web02 soft]# /soft/tomcat/bin/startup.sh
~~~

3.修改域名解析


***9.部署多节点Tomcat-->NFS***

10.0.0.7   ---> 10.0.0.51
10.0.0.8   ---> 10.0.0.51
				10.0.0.31

1.安装NFS
~~~
[root@nfs ~]# groupadd -g 666 www
[root@nfs ~]# useradd -u666 -g666 www
[root@nfs ~]# yum install nfs-utils -y
[root@nfs ~]# cat /etc/exports
/data/zrlog 172.16.1.0/24(rw,sync,all_squash,anonuid=666,anongid=666)
[root@nfs ~]# mkdir /data/zrlog
[root@nfs ~]# systemctl restart nfs
~~~
2.找到图片资源   推送图片资源至NFS
~~~
[root@web01 ~]# scp -rp /code/zrlog/ROOT/attached/* root@172.16.1.31:/data/zrlog/
[root@nfs ~]# chown -R www.www /data/zrlog/			#重新授权
~~~
3.多节点挂载  
~~~ 
# mount -t nfs 172.16.1.31:/data/zrlog/ /code/zrlog/ROOT/attached/
~~~



### 实现nginx负载均衡+tomcat

![1569588189472](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1569588189472.png)

