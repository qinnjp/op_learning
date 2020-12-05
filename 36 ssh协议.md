## ssh协议



***1.什么是SSH?***

SSH是一个安全协议，在进行数据传输时，会对数据包进行加密处理，加密后在进行数据传输。确保了数据传输安全。那SSH服务主要功能有哪些呢？

那么
ssh服务会对传输数据进行加密, 监听在本地22/tcp端口, ssh服务默认支持root用户登录
telnet服务不对数据进行加密, 监听在本地23/tcp端口, Telnet默认不支持root用户登录



***2.SSH主要的功能是?***

1.提供远程连接服务器的服务、
2.对传输的数据进行加密远程登录:
	SSH
	Telnet

除了SSH协议能提供远程连接服务，Telnet也能提供远程连接服务, 那么分别的区别是什么呢?



***3.SSH与Telnet之间有什么区别?***



| 服务连接方式 | 服务数据传输 | 服务监听端口 | 服务登陆用户         |
| ------------ | ------------ | ------------ | -------------------- |
| ssh          | 加密         | 22/tcp       | 默认支持root用户登陆 |
| telnet       | 明文         | 23/tcp       | 不支持root用户登陆   |



***4.抓包分析SSH与Telnet的区别?***

- 1.安装telnet服务并运行

```bash
[root@m01 ~]# yum install telnet-server -y
[root@m01 ~]# systemctl start telnet.socket
```

- 2.使用wireshark检测vmnet8网卡上telnet的流量

![img](https://img2018.cnblogs.com/blog/1488697/201905/1488697-20190528080614855-128893226.jpg)

- 3.telnet是无法使用root用户登录Linux系统，需要创建普通用户

```bash
[root@m01 ~]# useradd bgx
[root@m01 ~]# echo "123456"| passwd --stdin bgx
```

- 4.使用普通用户进行telnet登录

![img](https://img2018.cnblogs.com/blog/1488697/201905/1488697-20190528080628416-1863083057.jpg)

- 5.搜索wireshark包含telnet相关的流量

![img](https://img2018.cnblogs.com/blog/1488697/201905/1488697-20190528080702066-301158469.jpg)

![img](https://img2018.cnblogs.com/blog/1488697/201905/1488697-20190528080718743-1264603116.jpg)

![img](https://img2018.cnblogs.com/blog/1488697/201905/1488697-20190528080727476-1560728391.jpg)

服务器都是使用的SSH协议实现的远程登录
对于路由器  交换机  都是走的telnet协议  (  WEB界面调试  )

***5.SSH相关客户端指令ssh、scp、sftp?***



	1.ssh      ( Windows Xshell Crt )   ( Mac   ssh命令  Crt )
		[root@web01 ~]# ssh root@172.16.1.41
		root@172.16.1.41's password: 
	
	2.scp:   rsync增量    scp 全量(每次都是覆盖)  ssh协议
	拷贝目录 需要  -r参数
	推送
	[root@web01 ~]# scp ./web-file root@172.16.1.41:/tmp
	
	获取
	[root@web01 ~]# scp  root@172.16.1.41:/tmp/web-file  ./test
	
	限速 ( kb  1024 * 8 = 实际的传输速率 )
	[root@web01 ~]# scp -l 8192 ./1.txt 172.16.1.41:/tmp
	root@172.16.1.41's password: 
	1.txt                           14%   74MB   1.0MB/s   07:09 
	
	3.sftp 文件传输协议?
		为什么不适用命名的方式?  为什么使用xftp?
			1.简单,带图形,支持断点续传,支持暂停
***6.SSH远程登录方式、用户密码、秘钥方式?***

1.基于用户和密码的方式
		1.密码太复杂容易忘  lastpass
		2.密码太简单不安全
2.基于密钥的方式实现		(指纹)
	1.降低密码泄露风险
	2.提升用户的便捷性
	
3.实现免密码登录方式
	1.创建一对密钥   公钥+私钥 ==配套
	[root@manager ~]# ssh-keygen -C manager@qq.com
	.....一路回车.....

​	2.将管理机的公钥推送至web服务器上   ( 需要输入对端服务器的密码  )

​	[root@manager ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.16.1.7

​	3.使用 ssh 命令 连接 对应的服务器   ( 检查是否免密码  )
​	[root@manager ~]# ssh 'root@172.16.1.7'

​	4.有问题查看
​	tail -f /var/log/secure
​	https://www.jianshu.com/p/fb0df700305d	

***7.SSH场景实践，借助SSH免秘实现跳板机功能?***



***8.SSH远程连接功能安全优化? fail2ban又是啥?(研究)***

- 1.更改远程连接登陆的端口    	port 6666

- 2.禁止ROOT管理员直接登录		PermitRootLogin no
  		直接  xshell  -->root   --> server   (禁止用户名密码  禁止密钥)
  		间接  xshell  -->oldxu  --> server  ---> su - root

- 3.密码认证方式改为密钥认证		PasswordAuthentication no

- 4.重要服务不使用公网IP地址		!!!!!!!!!!!!!!!!!

- 5.使用防火墙限制来源IP地址		软件防火墙  |  硬件防火墙
  	
  		10.0.0.1(其他人)    --->  10.0.0.61		异常
  		10.0.0.100(公司)    --->  10.0.0.61	    正常

- 6.修改后的配置  [测试完后记得还原]

  ~~~bash
  [root@manager ~]# vim /etc/ssh/sshd_config
  	Port 6666                       # 变更SSH服务远程连接端口
  	PermitRootLogin         no      # 禁止root用户直接远程登录
  	PasswordAuthentication  no      # 禁止使用密码直接远程登录
  	UseDNS                  no      # 禁止ssh进行dns反向解析，影响ssh连接效率参数
  	GSSAPIAuthentication    no      # 禁止GSS认证，减少连接时产生的延迟
  ~~~

  域名解析IP 
  IP解析域名

***9.fail2ban又是啥?(研究)***		


ail2ban可以监控系统日志，并且根据一定规则匹配异常IP后使用Firewalld将其屏蔽，尤其是针对一些爆破/扫描等非常有效。

- 1.开启Firewalld防火墙
  [root@bgx ~]# systemctl start firewalld
  [root@bgx ~]# systemctl enable firewalld
  [root@bgx ~]# firewall-cmd --state
  running

- 2.修改firewalld规则，启用Firewalld后会禁止一些服务的传输，但默认会放行常用的22端口, 如果想添加更多，以下是放行SSH端口（22）示例，供参考：

~~~bash
#放行SSHD服务端口
[root@bgx ~]# firewall-cmd --permanent --add-service=ssh --add-service=http 
#重载配置
[root@bgx ~]# firewall-cmd --reload
#查看已放行端口
[root@bgx ~]# firewall-cmd  --list-service
~~~

- 3.安装fail2ban,需要有epel

~~~~bash
[root@bgx ~]# yum install fail2ban fail2ban-firewalld mailx -y
~~~~

- 4.配置fail2ban规则.local会覆盖.conf文件

~~~bash
[root@bgx fail2ban]# cat /etc/fail2ban/jail.local
[DEFAULT]
ignoreip = 127.0.0.1/8
bantime  = 86400
findtime = 600
maxretry = 5
banaction = firewallcmd-ipset
action = %(action_mwl)s

[sshd]
enabled = true
filter  = sshd
port    = 22
action = %(action_mwl)s
logpath = /var/log/secure
~~~



- 5.启动服务，并检查状态

~~~bash
[root@bgx ~]# systemctl start fail2ban.service
[root@bgx ~]# fail2ban-client status sshd
~~~





- 6.清除被封掉的IP地址

~~~bash
[root@bgx ~]# fail2ban-client set sshd unbanip 10.0.0.1
~~~



***10.SSH如何结合Google Authenticator 实现双向验证?  (适合自己用)***
	基于密码 + 动态口令		支持
	基于密钥 + 动态口令		不支持
	https://www.xuliangwei.com/bgx/1345.html


​	









​	
​	
​	