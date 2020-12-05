#### 1. 主从复制架构演变

读写分离: 读多写少

```bash 
Atlas , ProxySQL ,Maxscale .....
```

高可用: 解决业务主机宕机时,还能继续提供原有服务

```bash
MHA,PXC,MGC,MGR,MIC(8.0)
```

分布式架构: 将逻辑单元拆分到不同的节点中,分担存储压力和业务压力.

```bash
Mycat,DBLE,Sharding-jdbc
```

NewSQL架构: 合久必分,分久必合.

```bash
TiDB , Polardb ,TDSQL
```

#### 2. 企业高可用标准评估:全年无故障率

```bash
99.9%		0.1%		365*24*60*0.001= 525.6 min
99.99%		0.01%		365*24*60*0.001= 52.56 min
99.999% 	0.001%		365*24*60*0.001= 5.256 min
99.9999% 	0.0001%		365*24*60*0.001= 0.5256 min
```

负载均衡: 	3个9

主备集群: 	4个9	MHA

多活集群: 	5个9 	MySQL Cluster, MIC, PXC ,MGC

Real Application Cluster : Oracle  RAC   sysbase cluster 

#### 3. MHA环境基础

**3.1 规划**

主库: 51		node

从库:

- 52			node
- 53          node        manager

**3.2 准备环境（略。1主2从GTID）**

**配置关键程序软链接**

```bash
unlink /usr/bin/mysql
unlink /usr/bin/mysqlbinlog

ln -s /usr/local/mysql57/bin/mysqlbinlog    /usr/bin/mysqlbinlog
ln -s /usr/local/mysql57/bin/mysql          /usr/bin/mysql
```

**3.4 配置各节点互信(参考)**

db01:

```bash
rm -rf /root/.ssh 
ssh-keygen
cd /root/.ssh 
mv id_rsa.pub authorized_keys
scp  -r  /root/.ssh  10.0.0.52:/root 
scp  -r  /root/.ssh  10.0.0.53:/root 

#各节点验证
db01:
ssh 10.0.0.51 date
ssh 10.0.0.52 date
ssh 10.0.0.53 date

db02:
ssh 10.0.0.51 date
ssh 10.0.0.52 date
ssh 10.0.0.53 date

db03:
ssh 10.0.0.51 date
ssh 10.0.0.52 date
ssh 10.0.0.53 date
```

**3.5 安装软件**

- 1 下载软件

  ```bash
  mha官网：https://code.google.com/archive/p/mysql-master-ha/
  github下载地址：https://github.com/yoshinorim/mha4mysql-manager/wiki/Downloads
  ```

- 2 所有节点安装node软件依赖包

  ```bash
  yum install perl-DBD-MySQL -y
  rpm -ivh mha4mysql-node-0.56-0.el6.noarch.rpm
  ```

- 3 在db01主库中创建mha需要的用户

  ```sql
  grant all privileges on *.* to mha@'10.0.0.%' identified by 'mha';
  ```

- 4 Manager软件安装(db03)

  ```bash
  yum install -y perl-Config-Tiny epel-release perl-Log-Dispatch perl-Parallel-ForkManager perl-Time-HiRes
  rpm -ivh mha4mysql-manager-0.56-0.el6.noarch.rpm
  ```

**3.6 配置文件准备**(db03)

- 1 创建配置文件目录

  ```bash
  mkdir -p /etc/mha
  ```

- 2 创建日志目录

  ```bash
  mkdir -p /var/log/mha/app1
  ```

- 3 编辑mha配置文件

  ```bash
  cat > /etc/mha/app1.cnf <<EOF
  [server default]
  manager_log=/var/log/mha/app1/manager        
  manager_workdir=/var/log/mha/app1            
  master_binlog_dir=/data/binlog       
  user=mha                                   
  password=mha                               
  ping_interval=2
  repl_password=123
  repl_user=repl
  ssh_user=root                               
  [server1]                                   
  hostname=10.0.0.51
  port=3306                                  
  [server2]            
  hostname=10.0.0.52
  candidate_master=1
  port=3306
  [server3]
  hostname=10.0.0.53
  port=3306
  EOF
  ```

**3.7 状态检查**

- 1 互信检查

  ```bash
  masterha_check_ssh  --conf=/etc/mha/app1.cnf 
  ```

- 2 主从检查

  ```bash
  masterha_check_repl  --conf=/etc/mha/app1.cnf 
  ```

**3.8 开启MHA(db03)**

```bash
nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover  < /dev/null> /var/log/mha/app1/manager.log 2>&1 &
```

**3.9 查看MHA状态**

```bash
[root@db03 ~]# masterha_check_status --conf=/etc/mha/app1.cnf
app1 (pid:4719) is running(0:PING_OK), master:10.0.0.51

mysql -umha -pmha -h 10.0.0.51 -e "select @@server_id"
mysql -umha -pmha -h 10.0.0.52 -e "select @@server_id"
mysql -umha -pmha -h 10.0.0.53 -e "select @@server_id"
```

#### 5. MHA 软件结构介绍

Manager工具包主要包括以下几个工具：

| 工具                    | 作用                         |
| ----------------------- | ---------------------------- |
| masterha_manger         | 启动MHA                      |
| masterha_check_ssh      | 检查MHA的SSH配置状况         |
| masterha_check_repl     | 检查MySQL复制状况            |
| masterha_master_monitor | 检测master是否宕机           |
| masterha_check_status   | 检测当前MHA运行状态          |
| masterha_master_switch  | 控制故障转移（自动或者手动） |
| masterha_conf_host      | 添加或删除配置的server信息   |

Node工具包主要包括以下几个工具：
这些工具通常由MHA Manager的脚本触发，无需人为操作
save_binary_logs            保存和复制master的二进制日志 
apply_diff_relay_logs       识别差异的中继日志事件并将其差异的事件应用于其他的
purge_relay_logs            清除中继日志（不会阻塞SQL线程）

#### 6.MHA的工作原理 `*****`

- 1 通过masterha_manger脚本启动MHA高可用功能

- 2 通过masterha_master_monitor监控主节点的状态,每ping_interval秒探测一次主库的心跳,一共检测4次.如过.4次都没通,认为主库宕机

- 3 进行选主工作

  算法一: 权重candidate_master=1

  算法二: 判断日志量

  算法三: 按照配置文件的顺序

- 4 数据补偿

  ssh能连: 各个从库通过(save_binary_logs)立即保存缺失部分binlog到/var/tmp/xxxx,并补偿数据.
  ssh不能连:数据较少的从库,会通过apply_diff_relay_logs,计算差异,追平数据.

- 5 通过masterha_master_switch 脚本切换.所有从节点,stop slave; reset slave all;s2节点,change master to s1 ,start slave

- 6 调用masterha_conf_host脚本,从集群中将故障节点剔除.

- 7 manager 自杀.

有没有不足?还缺啥?
1. binlog server  日志冗余
2. 应用透明	vip漂移(keepalive,vip)	
3. 故障提醒
4. 自愈(待开发...),RDS,TDSQL都具备自愈能力.      	

#### 7. MHA 扩展功能应用

**7.1 MHA 的vip功能**

- 1 准备脚本

  ```bash
  [root@db03 ~]# cp master_ip_failover.txt /usr/local/bin/master_ip_failover
  [root@db03 ~]# cd /usr/local/bin/
  [root@db03 bin]# chmod +x master_ip_failover
  [root@db03 bin]# dos2unix master_ip_failover 
  
  #改配置文件:
  my $vip = '10.0.0.55/24';
  my $key = '1';
  my $ssh_start_vip = "/sbin/ifconfig eth0:$key $vip";
  my $ssh_stop_vip = "/sbin/ifconfig eth0:$key down";
  ```

- 2 manager配置文件修改

  添加参数

  ````bash
  vim  /etc/mha/app1.cnf
  master_ip_failover_script=/usr/local/bin/master_ip_failover
  ````

- 3 手工生成vip(主节点)

  ```bash
  [root@db01 ~]# ifconfig eth0:1 10.0.0.55/24
  ```

- 4 重启mha

  ```bash
  masterha_stop --conf=/etc/mha/app1.cnf
  
  nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/mha/app1/manager.log 2>&1 &
  
  masterha_check_status --conf=/etc/mha/app1.cnf
  
  ```

**7.2 binlog server（db03）**

*7.2.1 参数：*

```bash
vim /etc/mha/app1.cnf 
[binlog1]
no_master=1
hostname=10.0.0.53
master_binlog_dir=/data/mysql/binlog
```

*7.2.2 创建必要目录*

```bash
mkdir -p /data/mysql/binlog
chown -R mysql.mysql /data/*
```

*7.2.3  拉取主库binlog日志*

```bash
cd /data/mysql/binlog    
mysqlbinlog  -R --host=10.0.0.51 --user=mha --password=mha --raw  --stop-never mysql-bin.000001 &
```

*7.2.4 重启MHA*

```bash
masterha_stop --conf=/etc/mha/app1.cnf
nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/mha/app1/manager.log 2>&1 &
```

**7.3 邮件提醒**

*7.3.2 准备邮件脚本*
send_report

- 1 准备发邮件的脚本(上传 email_2019-最新.zip中的脚本，到/usr/local/bin/中)
- 2 将准备好的脚本添加到mha配置文件中,让其调用

*7.3.3. 修改manager配置文件，调用邮件脚本*

```bash
vi /etc/mha/app1.cnf
report_script=/usr/local/bin/send
```

- 3 停止MHA

  ```bash
  masterha_stop --conf=/etc/mha/app1.cnf
  ```

- 4 开启MHA

  ```bash
  nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/mha/app1/manager.log
  ```

- 5 关闭主库,看警告邮件 



#### 8. MHA 故障修复：

- 1 恢复故障节点db01

  ```bash
  /etc/init.d/mysqld start 
  ```

- 2 修复主从 

  将db01 加入到主从环境中作为从库角色 

  ```sql
  change master to 
  master_host='10.0.0.52',
  master_user='repl',
  master_password='123' ,
  MASTER_AUTO_POSITION=1;
  start slave;
  ```

- 3 修复binlog_server(db03)

  ```bash
  [root@db03 binlog]# cd /data/mysql/binlog/
  [root@db03 binlog]# rm -rf *
  [root@db03 binlog]#  mysqlbinlog  -R --host=10.0.0.52 --user=mha --password=mha --raw  --stop-never mysql-bin.000001 &
  ```

- 4 检查主库vip

  ```bash
  [root@db02 ~]# ifconfig -a
  ```

- 5 检查配置文件节点信息(db03)

  ```bash
  [server1]
  hostname=10.0.0.51
  port=3306
  
  [server2]
  hostname=10.0.0.52
  port=3306
  
  [server3]
  hostname=10.0.0.53
  port=3306
  ```

- 6 状态检查

  互信检查

  ```bash
  masterha_check_ssh  --conf=/etc/mha/app1.cnf 
  ```

  主从检查

  ```bash
  masterha_check_repl  --conf=/etc/mha/app1.cnf 
  ```

- 7 启动MHA 

  ```bash
  nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/mha/app1/manager.log 2>&1 &
  ```

## 读写分离Atlas+MHA应用

#### 1. 介绍

Atlas是由 Qihoo 360, Web平台部基础架构团队开发维护的一个基于MySQL协议的数据中间层项目。
它是在mysql-proxy 0.8.2版本的基础上，对其进行了优化，增加了一些新的功能特性。
360内部使用Atlas运行的mysql业务，每天承载的读写请求数达几十亿条。

下载地址
https://github.com/Qihoo360/Atlas/releases
注意：
1、Atlas只能安装运行在64位的系统上
2、Centos 5.X安装 Atlas-XX.el5.x86_64.rpm，Centos 6.X安装Atlas-XX.el6.x86_64.rpm。
3、后端mysql版本应大于5.1，建议使用Mysql 5.6以上

#### 2.安装配置

```bash
yum install -y Atlas*
cd /usr/local/mysql-proxy/conf
mv test.cnf test.cnf.bak
 vi test.cnf
[mysql-proxy]
admin-username = user
admin-password = pwd
proxy-backend-addresses = 10.0.0.55:3306
proxy-read-only-backend-addresses = 10.0.0.51:3306,10.0.0.53:3306
pwds = repl:3yb5jEku5h4=,mha:O2jBXONX098=
daemon = true
keepalive = true
event-threads = 8
log-level = message
log-path = /usr/local/mysql-proxy/log
sql-log=ON
proxy-address = 0.0.0.0:33060
admin-address = 0.0.0.0:2345
charset=utf8

#启动atlas
/usr/local/mysql-proxy/bin/mysql-proxyd test start
ps -ef |grep proxy
```

#### 3. Atlas功能测试

- 测试读操作：

  ```bash
  mysql -umha -pmha  -h 10.0.0.53 -P 33060 
  db03 [(none)]>select @@server_id;
  ```

- 测试写操作：

  ```sql
  mysql> begin;select @@server_id;commit;
  ```

#### 4. Atlas的管理

```sql
db03 [(none)]>select * from help;
+----------------------------+---------------------------------------------------------+
| SELECT * FROM help         | shows this help                                         |
| SELECT * FROM backends     | lists the backends and their state                      |
| SET OFFLINE $backend_id    | offline backend server, $backend_id is backend_ndx's id |
| SET ONLINE $backend_id     | online backend server, ...                              |
| ADD MASTER $backend        | example: "add master 127.0.0.1:3306", ...               |
| ADD SLAVE $backend         | example: "add slave 127.0.0.1:3306", ...                |
| REMOVE BACKEND $backend_id | example: "remove backend 1", ...                        |
| SELECT * FROM clients      | lists the clients                                       |
| ADD CLIENT $client         | example: "add client 192.168.1.2", ...                  |
| REMOVE CLIENT $client      | example: "remove client 192.168.1.2", ...               |
| SELECT * FROM pwds         | lists the pwds                                          |
| ADD PWD $pwd               | example: "add pwd user:raw_password", ...               |
| ADD ENPWD $pwd             | example: "add enpwd user:encrypted_password", ...       |
| REMOVE PWD $pwd            | example: "remove pwd user", ...                         |
| SAVE CONFIG                | save the backends to config file                        |
| SELECT VERSION             | display the version of Atlas                            |
+----------------------------+-----------------------------------

```

额外扩展:
ProxySQL
Maxscale
mysqlrouter
consul

Mycat 
优化 
redis
mongodb











