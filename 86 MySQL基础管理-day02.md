## 第二章 基础管理

#### 1. 用户管理

**1.1 作用**

登录

管理对象(库表)

**1.2 定义**

用户名@'白名单'

*1.2.1用户名:*

不能太长,和业务有关,开头必须是字母.

emp_user01

```bash
grant all on *.* to wordpress@'%' identified '123';
```

*1.2.2 白名单*

```bash
`常用`
wordpress@'10.0.0.%'    	#允许wordpress通过10.0.0.0/24访问
wordpress@'10.0.0.0/255.255.254.0' #
wordpress@'localhost'		#本地
wordpress@'10.0.0.5%'		#50-59

wordpress@'%'				#任意
wordpress@'10.0.0.200'		#只允许10.0.0.200连接
wordpress@'db02'			
```

*1.2.3 用户管理* 

```bash
#创建用户:
mysql> CREATE USER  oldguo@'10.0.0.%' IDENTIFIED BY '123';

#查询用户:(mysql下面所有用户都存在一张表中)
mysql> select user,host from mysql.user;
##带密码
mysql> select user,host,authentication_string  from mysql.user;

#修改用户:
mysql> alter user oldguo@'10.0.0.%' identified by '123456';

#删除用户: 
mysql> drop user oldguo@'10.0.0.%';
```

`8.0以前可以创建用户时授权,8.0以后必须先创建用户在授权`

   

#### 2. 权限管理

**2.1 MySQL权限列表**

```bash
show privileges;
```

all代表除去Grant option 的所有权限

**2.2 授权**

`GRANT  权限   ON 权限作用范围   TO 用户   IDENTIFIED BY '123' with grant option;`



- 权限

  all:

  ```bash
  Alter                  
  Alter routine          
  Create                 
  Create routine         
  Create temporary tables
  Create view            
  Create user            
  Delete                 
  Drop                   
  Event                  
  Execute                
  File                   
  #Grant option           		#给其他用户授权
  Index                  
  Insert                 
  Lock tables            
  Process                
  Proxy                  
  References             
  Reload                 
  Replication client     
  Replication slave      
  Select                 
  Show databases         
  Show view              
  Shutdown               
  Super                  
  Trigger                
  Create tablespace      
  Update                 
  Usage
  ```

- 权限的作用范围:

  ```bash
  *.*			--->对所有库表有权限
  oldduo.* 	--->对oldguo库有权限
  oldguo.t1	--->对oldguo库下的t1表有权限
  ```

  

**2.3 企业授权案例**

(1)授权一个管理员用户oldguo,可以从10网段任意地址登录管理数据库 

```bash
GRANT  ALL  ON *.*    TO oldguo@'10.0.0.%'   IDENTIFIED BY '123'  with grant option;
```

(2)授权一个业务用户app,可以从10网段地址访问app库的所有表 

```bash
grant select,update,insert,delete ON app.* TO app@'10.0.0.%'   IDENTIFIED BY '123' ;
```

(3)授权一个开发用户dev,可以对dev库进行业务开发

**2.4 root管理员密码忘记或被篡改如何处理?**

(1) 关闭数据库,启动到"单用户"模式

```bash
[root@db01 data_3306]# systemctl stop mysqld
[root@db01 data_3306]# mysqld_safe  --skip-grant-tables  --skip-networking  &

--skip-grant-tables		#关闭用户验证功能
--skip-networking		#关闭远程登录
```

(2) 无密码登录MySQL

```bash
[root@db01 data_3306]# mysql
mysql> alter user root@'localhost' identified by '123456';
ERROR 1290 (HY000): The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement

mysql> flush privileges;
mysql> alter user root@'localhost' identified by '123456';
```

(3) 重启数据库到正常模式

```bash
[root@db01 data_3306]# systemctl restart mysqld
```

**2.5 查询用户权限**

```bash
mysql> show grants for app@'10.0.0.%';
```

**2.6 回收权限**

```bash
mysql> revoke delete,drop on app.* from 'app'@'10.0.0.%';
```

#### 3.MySQL的连接管理

**3.1 自带客户端工具**

*3.1.1 mysql*

| 选项 | 含义        |
| ---- | ----------- |
| -u   | 用户名      |
| -p   | 密码        |
| -h   | IP          |
| -P   | 端口        |
| -S   | socket位置  |
| -e   | 免交互      |
| <    | 导入SQL脚本 |

例子:

(1) TCP连接串远程登录

注:需要提前创建好用户

```bash
mysql> grant all on *.* to oldguo@'10.0.0.%' identified by '123';
[root@db01 data_3306]# mysql -uoldguo -p -h 10.0.0.51 -P 3306
Enter password:

```

(2) Socket连接方式

注: 需要提前创建好localhost用户

```bash
mysql> grant all on *.* to oldguo@'localhost' identified by '123';
[root@db01 data_3306]# mysql -uoldguo -p -S /tmp/mysql.sock 
Enter password:
```

(3) 免交互执行命令

```bash
mysql -uroot -p -e "show privileges"
```

*3.1.2 mysqladmin*

(1) 修改密码

```bash
mysqladmin -uroot -p 123456 password 123
```

(2) 关闭数据库

```bash
mysqladmin -uroot -p123 shutdown
```

mysqldump

**3.2 第三方开发工具**

sqlyog

navicat

workbench

**3.3 应用程序连接**

php-mysql 
pip3 install mysql 
jar 
go 

#### 4. MySQL的启动关闭

systemctl ---> mysql.server start -----> mysqld_safe  ----> mysqld

- 使用mysqld启动数据库

```bash
mysqld_safe & 		#直接启动数据库
mysql &
```

#### 5. MySQL的初始化配置

**5.1 初始化配置方法**
源码安装定制    <    初始化配置文件  <    命令行启动时定制

**5.2 初始化配置文件**

```bash
[root@db01 data_3306]# mysqld --help --verbose |grep my.cnf
/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf 
```

建议一个mysql实例一个配置文件

**5.3 配置文件书写格式**

```bash
[root@db01 data_3306]# cat /etc/my.cnf 
[mysqld]
user=mysql
port=3306
basedir=/usr/local/mysql57
datadir=/data/mysql/data_3306
server_id=6
socket=/tmp/mysql.sock
[mysql]
socket=/tmp/mysql.sock
```

标签项  ====> [mysqld]

服务器端 [server]: [mysqld],[mysqld_safe]  ====> 影响到MySQL启动

客户端 [clinet] : [mysql] ,[mysqldump]    ====> 影响本地客户端程序



配置项  ====> key=value

#### 6. 多实例的规划和配置

分布式架构中应用广泛

**6.1 端口和目录**

```bash
rm -rf /data/mysql/data_{3307,3308,3309}
mkdir -p /data/mysql/data_{3307,3308,3309} 
```

**6.2 配置文件准备** 

```bash
cat > /data/mysql/my3307.cnf <<EOF
[mysqld]
user=mysql
port=3307
basedir=/usr/local/mysql57
datadir=/data/mysql/data_3307
server_id=7
socket=/tmp/mysql3307.sock
EOF
cat > /data/mysql/my3308.cnf <<EOF
[mysqld]
user=mysql
port=3308
basedir=/usr/local/mysql57
datadir=/data/mysql/data_3308
server_id=8
socket=/tmp/mysql3308.sock
EOF
cat > /data/mysql/my3309.cnf <<EOF
[mysqld]
user=mysql
port=3309
basedir=/usr/local/mysql57
datadir=/data/mysql/data_3309
server_id=9
socket=/tmp/mysql3309.sock
EOF
```

**6.3 授权** 

```bash
[root@db01 ~]# chown -R mysql.mysql /data/
```

**6.4 初始化数据** 

```bash
mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql57 --datadir=/data/mysql/data_3307
mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql57 --datadir=/data/mysql/data_3308
mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql57 --datadir=/data/mysql/data_3309
```

**6.5 启动多实例** 

```bash
[root@db01 mysql]# mysqld --defaults-file=/data/mysql/my3307.cnf &
[root@db01 mysql]# mysqld --defaults-file=/data/mysql/my3308.cnf &
[root@db01 mysql]# mysqld --defaults-file=/data/mysql/my3309.cnf &
[root@db01 mysql]# netstat -tulnp
```

**6.6 使用 systemd 管理多实例**

```nash
cat >/etc/systemd/system/mysqld3307.service <<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/usr/local/mysql57/bin/mysqld --defaults-file=/data/mysql/my3307.cnf
LimitNOFILE = 5000
EOF

cat >/etc/systemd/system/mysqld3308.service <<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/usr/local/mysql57/bin/mysqld --defaults-file=/data/mysql/my3308.cnf
LimitNOFILE = 5000
EOF

cat >/etc/systemd/system/mysqld3309.service <<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/usr/local/mysql57/bin/mysqld --defaults-file=/data/mysql/my3309.cnf
LimitNOFILE = 5000
EOF

pkill mysqld  
systemctl start mysqld3307
systemctl start mysqld3308
systemctl start mysqld3309
```



