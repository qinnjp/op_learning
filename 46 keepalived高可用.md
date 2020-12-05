## keepalived高可用

***昨日练习***

*   需求: 公司网站在停机维护时，指定的IP能够正常访问，其他的IP跳转到维护页。10.0.0.1 10.0.0.100
~~~bash
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
* 需求：公司网站后台/admin，只允许公司的出口公网IP可以访问，其他的IP访问全部返回500，或直接跳转至首页。
~~~bash
server {
 listen 80;
 server_name ds.oldxu.com;
 root /data;
 set $ip 0;
 if ($remote_addr ~* "10.0.0.1") {
 set $ip 1;
 }

 location /admin {
 if ($ip = 0) {
 rewrite ^(.*)$  http://ds.oldxu.com;
 #return 500;
 }
 index index.html; 
 }

}
~~~

***1.什么是高可用，为什么要设计高可用？***

简单理解：两台机器启动着相同的业务系统，当有一台机器宕机，另外一台能快速接管，对于访问的用户是无感知的。

专业理解：高可用是分布式系统架构设计中必要的一环，主要目的：减少系统不能提供服务的时间。

***2.高可用使用什么工具来实现？是硬件还是软件？***

软件keepalive

***3.keeplived是如何实现高可用的？***

keeplived软件是基于VRRP协议实现的。VRRP虚拟路由冗余协议，主要用于解决单点故障问题。

***4.那么VRRP是如何诞生的，VRRP的原理又是什么？***

比如公司的网络是通过网关转换进行上网的那么如果该路由器故障了，网关无法转发报文了此时所有人都将无法上网，这时候怎么办？

![image](https://upload-images.jianshu.io/upload_images/18911567-9eb92593d114c937.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

通常做法是给路由增加一台备节点，但问题来了?如果我们的主网关master故障了，用户是需要手动修改网关指向Backup，如果用户过多修改起来会非常的麻烦。  第一个问题: 假设用户将指向都修改到Backup路由器，那么Master路由器如果修复好了又该怎么办？  第二个问题: 假设Master网关故障，我们将Backup网关配置为Master网关IP行不行?  其实上不行，因为PC第一次是通过ARP广播寻找到Master网关的Mac地址与IP地址，PC则会将Master网关的对应IP与MAC地址写入ARP缓存表中，那么PC第二次则会直接读取ARP缓存表中的MAC地址与IP地址，然后进行数据包的转发。此时PC转发的数据包还是会教给Master。(除非PC的ARP缓存表过期，在次发起ARP广播的时候才能正确获取Bakcup的Mac地址与对应的IP地址。)

![image](https://upload-images.jianshu.io/upload_images/18911567-33ca1f28ea32ad15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

如何才能做到出现故障自动转移，此时VRRP就应运而生，我们的VRRP其实是通过软件或硬件的形式在Master和Backup外面增加一个虚拟MAC地址(简称VMAC)与虚拟IP地址(简称VIP)。那么在这种情况下，PC请求VIP的时候，无论是Master处理还是Backup处理，PC仅会在ARP缓存表中记录VMAC与VIP的对应关系。

![image](https://upload-images.jianshu.io/upload_images/18911567-defdee06b536c70b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

***5.高可用keepalived使用场景***

通常业务系统需要保证7x24小时不DOWN机, 比如公司内部OA系统，每天公司人员都需要使用，则不允许Down机。作为业务系统来说随时都可用

![image](https://upload-images.jianshu.io/upload_images/18911567-5a79d57db059fc20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

***1.6.高可用核心概念总结***

*   1.如何确定谁是主节点谁是备节点。(投票选举？优先级？)

*   2.如果Master故障，Backup自动接管，那Master恢复后会夺权吗？(抢占式、非抢占式)

*   3.如果两台服务器都认为自己是Master会出现什么问题？(脑裂)

***7.keepalived高可用安装与配置?***

10.0.0.5  10.0.0.6

*   1.安装
~~~bash
yum install keepalived -y</pre>
~~~
*   2.配置
~~~bash
#抢占式
[root@lb01 ~]# cat /etc/keepalived/keepalived.conf 
global_defs { 
 router_id lb01 
}

vrrp_instance VI_1 {
 state MASTER
 interface eth0
 virtual_router_id 50
 priority 150
 advert_int 1
 authentication {
 auth_type PASS
 auth_pass 1111
}
 virtual_ipaddress {
 10.0.0.3
 }
}
#############################################
[root@lb02 ~]# cat /etc/keepalived/keepalived.conf global_defs {
 router_id lb02
}

vrrp_instance VI_1 {
 state BACKUP
 interface eth0
 virtual_router_id 50
 priority 100
 advert_int 1
 authentication {
 auth_type PASS
 auth_pass 1111
 }
 virtual_ipaddress {
 10.0.0.3
 }
}
~~~

*   3.启动
~~~bash
[root@lb01 ~]# systemctl start keepalived
[root@lb01 ~]# systemctl enable keepalived</pre>
~~~
*   4.验证
~~~bash
[root@lb01 ~]# ip addr|grep 10.0.0.3</pre>
~~~
***8.keepalived高可用地址漂移测试?***

*   在命令提示符ping 10.0.0.3抓包分析

当停止10.0.0.5上的keepalived是虚拟ip10.0.0.3跳转至10.0.0.6上
![image.png](https://upload-images.jianshu.io/upload_images/18911567-ad0ae954fe0f8c43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



当恢复10.0.0.5上的keepalived是又跳转回来
![image.png](https://upload-images.jianshu.io/upload_images/18911567-caf54ac59ea975d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


*   arp分析

当主机10.0.0.5正常的时候虚拟ip10.0.0.3的mac地址与主机10.0.0.5的mac地址一致

![image.png](https://upload-images.jianshu.io/upload_images/18911567-b5a6b520f92ba94d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


当主机故障时虚拟ip10.0.0.3的mac地址与主机10.0.0.6的mac地址一致，主机恢复后又回到主机
![image.png](https://upload-images.jianshu.io/upload_images/18911567-911eb0ad50a934da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


***9.keepalived*高可用抢占式与非抢占式***  
1.master故障--->backup顶上--->master恢复--->backup 抢占式 默认  2.master故障--->backup顶上--->master恢复--->backup继续工作  非抢占式

1、两个节点的state都必须配置为BACKUP(官方建议)  2、两个节点都在vrrp_instance中添加nopreempt参数  3、其中一个节点的优先级必须要高于另外一个节点的优先级。  两台服务器都角色状态启用nopreempt后，必须修改角色状态统一为BACKUP，唯一的区分就是优先级。

1.抢占: 硬件配置不一  2.非抢占: 硬件配置一致,业务不允许多次切换

***10.keepalived高可用与Nginx集成***  地址漂移实现高可用  nginx和keeplaived没有关系?  nginx需要借助keeplaived VIP 地址漂移 实现 高可用.

***11.如果Nginx宕机, 会导致用户请求失败, 但Keepalived并不会进行切换, 所以需要编写一个脚本检测Nginx的存活状态, 如果不存活则kill nginx和keepalived***

1.判断nginx进程是否存在  ps aux|grep nginx|grep -v grep  2.判断nginx的端口是否存在  netstat -lntp|grep :80|wc -l  3.通过curl来模拟访问,判断访问结果是否ok curl -H Host:url.oldxu.com [http://10.0.0.3](http://10.0.0.3)

*   1.编写脚本
~~~bash
[root@lb01 ~]# mkdir /scripts
[root@lb01 ~]# vim /scripts/check_web.sh
#!/usr/bin/bash

nginx_port=$(netstat -lntp|grep :80|wc -l)
if [ $nginx_port -ne 1 ];then

 systemctl start nginx &>/dev/null
 rc=$?
 sleep 3
 if [ $rc -ne 0 ];then
 systemctl stop  keepalived 
 fi
fi
[root@lb01 ~]# chmod +x /scripts/check_web.sh </pre>
~~~
*   2.keeplaived调用该脚本
~~~bash
[root@lb01 ~]# cat /etc/keepalived/keepalived.conf 
 global_defs { 
 router_id lb01 
 }

 #定义脚本名称,以及脚本所在的路径
 vrrp_script check_web {
 script "/scripts/check_web.sh"
 interval 5
 }


 vrrp_instance VI_1 {
 state MASTER
 #nopreempt
 interface eth0
 virtual_router_id 50
 priority 150
 advert_int 1
 authentication {
 auth_type PASS
 auth_pass 1111
 }
 virtual_ipaddress {
 10.0.0.3
 }

 #调用脚本名称
 track_script {
 check_web
 }
 }
~~~
*   3.模拟nginx停止,检查nginx是否会被拉起

*   4.模拟nginx故障,检查keeplaived的VIP是否会漂移至备节点

***11.keepalived高可用脑裂与故障解决?***

脑裂（split-brain），指在一个高可用（HA）系统中，当联系着的两个节点断开联系时，本来为一个整体的系统，分裂为两个独立节点，这时两个节点开始争抢共享资源，结果会导致系统混乱，数据损坏。

对于无状态服务的HA，无所谓脑裂不脑裂；  但对有状态服务(比如MySQL)的HA，必须要严格防止脑裂。  （但有些生产环境下的系统按照无状态服务HA的那一套去配置有状态服务，结果可想而知...）

master 10.0.0.3  backup 10.0.0.3  1.在备上编写检测脚本, 测试如果能ping通主并且备节点还有VIP的话则认为产生了脑列
~~~bash
[root@lb02 conf.d]# cat /scripts/check_spilt.sh 
vip=10.0.0.3
master_ip=10.0.0.5

ping -c2 $master_ip &>/dev/null
if [ $? -eq 0 ];then
 ip_check=$(ip addr | grep "$vip" | wc -l)
 if [ $ip_check -eq 1 ];then
 echo "脑列"
 systemctl stop keepalived
 fi
fi
[root@lb02 conf.d]# cat /etc/keepalived/keepalived.conf 
global_defs {
 router_id lb02
}

vrrp_script check_spilt {
 script "/scripts/check_spilt.sh"
 interval 3
}


vrrp_instance VI_1 {
 state BACKUP
 nopreempt
 interface eth0
 virtual_router_id 50
 priority 100
 advert_int 1
 authentication {
 auth_type PASS
 auth_pass 1111
 }
 virtual_ipaddress {
 10.0.0.3
 }

 track_script {
 check_spilt
 }

}

~~~
keeplaived使用:  1.不能在公有云上使用  2.公有云要想实现负载均衡高可用-----> 购买的SLB 自带高可用  3.虚拟IP咋使--->真实的硬件环境:
![image.png](https://upload-images.jianshu.io/upload_images/18911567-ef99c5a4faebd716.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)