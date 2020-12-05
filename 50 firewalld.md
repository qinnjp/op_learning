## 1.本周内容

* firewalld防火墙
* ansible自动化配置工具 (安装 配置 启动 )

## 2.今日内容

 * 1.题目--->
 * 2.安全
 * 3.firewalld防火墙 (软件型)
 * 4.firewalld防火墙-->区域
 * 5.firewalld放行端口相关
 * 5.firewalld放行服务相关
 * 6.firewalld富规则-->
 * 7.firewalld实现共享上网

## 3.Firewalld

### 3.1 安全

* 硬件环境:
  * 硬件层面:  电源 (UPS)  温度监控  机柜上锁   磁盘报警
  * 系统层面:  
    * 更换默认SSH端口 
    * 禁止ROOT直接登录,统一使用密钥认证方式
    * 使用防火墙限制-->某个来源IP才能连接SSH
    * 软件更新  内核升级   --->已运行很久系统不要升级内核
  * 服务: mysql  redis  等等
    * 不要有公网IP地址 
    * 如果有公网IP,不要监听在0.0.0.0
    * 一定要设定比较复杂的密码认证
  * web: nginx tomcat  应用层
    * HTTPS
    * WAF -->web应用防火墙       (防火墙+WAF防火墙)   --> http/https协议
      * 安全宝   牛盾云    安全狗     知道创宇    阿里云  

* 云环境:
  * 系统层面:   
    * SSH
    * 安骑士(免费版)      云安全中心(收费版)
    * 快照  --->  使用快照需要购买存储空间
  * 服务层面:  redis  mysql 
    - 不要有公网IP地址 
    - 如果有公网IP,不要监听在0.0.0.0
    - 一定要设定比较复杂的密码认证
    - 安全组(防火墙)
  * web层面:
    * HTTPS
    * 云WAF    
  * 数据层面:
    * 备份
    * 异地备份
  * 上网  --->VPC --->NAT网关       (端口映射)

找几家做安全公司,    ngi nx+lua

* 云架构
  * 高防IP                 --->DDOS
  * WAF防火墙       --->漏洞注入
  * HTTPS                 ---->防劫持 防篡改

HTTPS+WAF+负载均衡

<https://help.aliyun.com/document_detail/61993.html?spm=a2c4g.11186623.6.573.c85f5414Ajl83H>

![1570502634323](标杆班级-ansible.assets/1570502634323.png)



HTTPS+高仿IP+WAF+负载均衡

<https://help.aliyun.com/document_detail/35163.html?spm=a2c4g.11186623.6.792.2fc36251vRqVB1>

高防需要配置HTTPS    -->      WAF也需要配置HTTPS      -->  源站 http

**考虑安全  性能差 **

**考虑性能 安全弱**

![1570499089835](标杆班级-ansible.assets/1570499089835.png)

### 3.2 Firewalld防火墙

 * firewalld  --->   无需网络知识		--->     自动挡汽车      不支持花活

 * iptables    --->   依赖网络知识               ---->    手动挡汽车     花活

   firewalld比iptables简单--->

    * 图形界面操作   GUI     太复杂
    * 命令行操作         CLI     简单

 * 80 22 3306

* 一个网卡仅能绑定一个区域。比如: eth0-->A区域 

* 但一个区域可以绑定多个网卡。比如: B区域-->eth0、eth1、eth2

* 还可以根据来源的地址设定不同的规则。比如：所有人能访问80端口，但只有公司的IP才允许访问22端口。



firewalld查看处于哪个区域

```bash
[root@manager ~]# firewall-cmd --get-active-zones 
public

[root@manager ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0 eth1
  sources: 
  services: ssh dhcpv6-client
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 

```

*3.使用firewalld各个区域规则结合配置，调整默认public区域拒绝所有流量，但如果来源IP是10.0.0.0/24网段则允许。     复规则实现(只需要一个区域)

```bash
[root@manager ~]# firewall-cmd --remove-service={ssh,dhcpv6-client}
[root@manager ~]# firewall-cmd --add-source="10.0.0.0/24" --zone=trusted
[root@manager ~]# firewall-cmd --get-active-zones 
public
  interfaces: eth0 eth1
trusted
  sources: 10.0.0.0/24
  

#清空配置
[root@manager ~]# firewall-cmd --reload

```

*4.firewalld放行端口*

```bash
#添加放行端口
[root@manager ~]# firewall-cmd --add-port=80/tcp
[root@manager ~]# firewall-cmd --add-port={8081/tcp,8082/tcp}

#移除放行端口
[root@manager ~]# firewall-cmd --remove-port=80/tcp 
[root@manager ~]# firewall-cmd --remove-port={8081/tcp,8082/tcp}

[root@manager ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0 eth1
  sources: 
  services: ssh dhcpv6-client
  ports: 80/tcp 8081/tcp 8082/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

*5.放行服务--->对应的还是端口-->比端口看起来更清晰*

```bash
[root@manager ~]# firewall-cmd --add-service=http
[root@manager ~]# firewall-cmd --add-service=https
[root@manager ~]# firewall-cmd --add-service={zabbix-agent,zabbix-server}
[root@manager ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0 eth1
  sources: 
  services: ssh dhcpv6-client http https zabbix-agent zabbix-server
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
  
  
 移除
[root@manager ~]# firewall-cmd --remove-service={zabbix-agent,zabbix-server}
```

4.自定义服务名称--->服务对应的端口       8080  8081 8082 -->api业务

```bash
[root@manager services]# cd /usr/lib/firewalld/services
[root@manager services]# cp http.xml api.xml
[root@manager services]# cat api.xml 
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>API (HTTP)</short>
  <port protocol="tcp" port="8081"/>
  <port protocol="tcp" port="8082"/>
  <port protocol="tcp" port="8083"/>
</service>

[root@manager services]# firewall-cmd --reload
success
[root@manager services]# firewall-cmd --add-service=api
success
```

*5.firewalld实现端口转发*

* 路由器  ---------->TPLINK

* 虚拟机Vmware

* Nginx四层TCP/IP      (类似       和lvs不一样)

* Firewalld 

  ![1570518954254](标杆班级-ansible.assets/1570518954254.png)

  ```bash
  firewall-cmd --permanent --zone=<区域> --add-forward-port=port=<源端口号>:proto=<协议>:toport=<目标端口号>:toaddr=<目标IP地址>
  
  [root@manager ~]# firewall-cmd --add-forward-port=port=5555:proto=tcp:toport=22:toaddr=172.16.1.31
  
  # 地址伪装
  [root@manager ~]# firewall-cmd --add-masquerade
  #------------------------------------------------------->
  #管理上抓包
  [root@manager ~]# tcpdump port 5555 -nn
  
  #后端主机的抓包
  [root@nfs ~]# tcpdump -i eth1 port 22 -nn
  ```

*6.Firewalld富规则*

```
[root@Firewalld ~]# man firewalld.richlanguage  # 获取富规则手册
    rule
        [source]
        [destination]
        service|port|protocol|icmp-block|masquerade|forward-port
        [log]
        [audit]
        [accept|reject|drop]

rule [family="ipv4|ipv6"]
source address="address[/mask]" [invert="True"]
service name="service name"
port port="port value" protocol="tcp|udp"
forward-port port="port value" protocol="tcp|udp" to-port="port value" to-addr="address"
accept | reject [type="reject type"] | drop
```

*1.允许10.0.0.1主机能够访问    http服务，允许172.16.1.0/24能访问8080端口*

```bash
10.0.0.1   --->    80         http     除此以外都不可以访问
172.16.1.x --->    8080       api      除此以外都不可以访问

[root@manager ~]# firewall-cmd --reload

[root@manager ~]# firewall-cmd --add-rich-rule='rule family=ipv4  source address=10.0.0.1/32 service name="http" accept'

[root@manager ~]# firewall-cmd --add-rich-rule='rule family=ipv4  source address=172.16.1.0/24 service name="api" accept'
```

*2.默认public区域对外开放所有人能通过ssh服务连接，*但拒绝172.16.1.0/24网段通过ssh连接服务器*

```bash
[root@manager ~]# firewall-cmd --add-rich-rule='rule family=ipv4 source address=172.16.1.0/24 service name="ssh" drop'
```

*3.使用firewalld，允许所有人能访问http,https服务，但只有10.0.0.1主机可以访问ssh服务*

```bash
[root@manager ~]# firewall-cmd --reload
[root@manager ~]# firewall-cmd --add-service={http,https}
[root@manager ~]# firewall-cmd --remove-service=ssh

[root@manager ~]# firewall-cmd --add-rich-rule='rule family=ipv4 source address=10.0.0.1/32 service name="ssh" accept'


####使用端口方式  2222
[root@manager ~]# firewall-cmd --add-rich-rule='rule family=ipv4 source address=10.0.0.1/32 port port="2222" protocol=tcp accept'
```

*4.当用户来源IP地址是10.0.0.1主机，则将用户请求的5555端口转发至后端172.16.1.31的22端口*

```bash
任何人访问10.0.0.61 5555端口都给转发
10.0.0.1   --->    10.0.0.61 5555   ---> 172.16.1.31 22

[root@manager ~]# firewall-cmd --add-rich-rule='rule family=ipv4 source address="10.0.0.1/32" forward-port port="5555" protocol="tcp" to-port="22" to-addr="172.16.1.31"'
[root@manager ~]#  firewall-cmd --add-masquerade 
```



***7.firewalld实现共享上网***

*在指定的带有公网IP的实例上启动Firewalld防火墙的NAT地址转换，以此达到内部主机上网。*

![img](标杆班级-ansible.assets/15442375496785.jpg)



详细流程

![img](标杆班级-ansible.assets/15517748885644.jpg)



*1.firewalld防火墙开启masquerade, 实现地址转换*

```bash
[root@Firewalld ~]# firewall-cmd --add-masquerade --permanent
[root@Firewalld ~]# firewall-cmd --reload
```

*2.客户端将网关指向firewalld服务器，将所有网络请求交给firewalld*

```bash
[root@web03 ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth1
GATEWAY=172.16.1.61
```

*3.客户端还需配置dns服务器*

```bash
[root@web03 ~]# cat /etc/resolv.conf
nameserver 223.5.5.5
```

*4.重启网络，使其配置生效*

```bash
[root@web03 ~]# ifdown eth1 && ifup eth1
```

*5.测试后端web的网络是否正常*

```bash
[root@web03 ~]# ping baidu.com
PING baidu.com (123.125.115.110) 56(84) bytes of data.
64 bytes from 123.125.115.110 (123.125.115.110): icmp_seq=1 ttl=127 time=9.08 ms
```

firewalld共享上网方式不是特别推荐     (阿里云上如何实现---> 提交工单)

推荐:   

​	物理环境: 使用路由器来实现上网

​	云环境:    推荐使用NAT网关设备

-------------------------------------------------------------------------------------------------------------------------------------

安全体系:  OSI七层模型  --->   

云架构 (      高防IP  +  WAF防火墙    )               |  单机架构|   集群架构  |多机房  | SOA   

  WAF+负载均衡配置        

* 1.配置负载均衡+多web节点 
* 2.配置WAF    --->   指向负载均衡的公网IP地址       --->完成后会生成一个cname的域名.
* 3.配置DNS   解析---> 指向WAF   cname
* 4.配置HTTPS,请将证书传一份至WAF上,  至于负载均衡需不需要看情况.

高防IP  +  WAF防火墙 + 负载均衡

Firewalld 默认是拒绝所有端口

​	区域概念

​	放行端口

​	放行服务

​	端口转发

​	共享上网

​	富规则



作业:

​	1.问答题   (20~40)

​	2.前后端项目

​	3.Ansible

