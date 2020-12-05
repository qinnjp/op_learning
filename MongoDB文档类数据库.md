## MongoDB文档类数据库

**概述** 

- 适合存储大量,不怎么重要的数据.

- 存储json格式的数据

- 最接近关系型数据库的NoSQL,自带高可用,分布式集群.

最初使用在存储地图方面的数据,如滴滴等.使用MongoDB最多的公司360.

#### 1.MongoDB的逻辑结构

库database

集合collection

文档document

#### 2.安装部署

**1.系统准备**

- 1.Redhat或centos6.2以上系统
- 2.系统开发包完整

- 3.IP地址和hosts文件解析正常

- 4.iptables防火墙&selinux关闭

- 5.关闭大页内存机制

  root用户下

  ```bash
  vi /etc/rc.local
  if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
    echo never > /sys/kernel/mm/transparent_hugepage/enabled
  fi
  if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
     echo never > /sys/kernel/mm/transparent_hugepage/defrag
  fi
  #添加在最后 
   
  cat  /sys/kernel/mm/transparent_hugepage/enabled     
  cat /sys/kernel/mm/transparent_hugepage/defrag 
  
  echo never > /sys/kernel/mm/transparent_hugepage/enabled
  echo never > /sys/kernel/mm/transparent_hugepage/defrag
  ```

其他系统关闭参照官方文档:

https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/

为什么要关闭大页内存?

```bash
Transparent Huge Pages (THP) is a Linux memory management system that reduces the overhead of Translation Lookaside Buffer (TLB) lookups on machines with large amounts of memory by using larger memory pages.However, database workloads often perform poorly with THP, because they tend to have sparse rather than contiguous memory access patterns. `You should disable THP on Linux machines to ensure best performance with MongoDB.`
关闭大页内存(THB)可以是MongoDB具有更好的性能
```

#### 2.MongoDB安装

1.创建用户和组

```bash
useradd mongod
passwd mongod
```

2.创建MongoDB所需目录结构

```bash
mkdir -p /mongodb/conf
mkdir -p /mongodb/log
mkdir -p /mongodb/data
```

3.上传并解压软件到指定位置

```bash
cd   /data
tar xf mongodb-linux-x86_64-rhel70-3.6.12.tgz 
cp -r /data/mongodb-linux-x86_64-rhel70-3.6.12/bin/ /mongodb
```

4.设置目录权限

```bash
chown -R mongod:mongod /mongodb
```

5.设置用户环境变量

```bash
su - mongod

vi .bash_profile
export PATH=/mongodb/bin:$PATH

source .bash_profile
```

6.启动MongoDB

```bash
mongod --dbpath=/mongodb/data --logpath=/mongodb/log/mongodb.log --port=27017 --logappend --fork 

--logappend #已追加方式是存储日志,默认是覆盖方式

```

7.登录MongoDB

```bash
[mongod@server2 ~]$ mongo
```

8.使用配置文件

yaml模式

```yaml
NOTE：
YAML does not support tab characters for indentation: use spaces instead.

#系统日志有关  
systemLog:
   destination: file        
   path: "/mongodb/log/mongodb.log"    #日志位置
   logAppend: true                     #日志以追加模式记录
  
#数据存储有关   
storage:
   journal:
      enabled: true
   dbPath: "/mongodb/data"            #数据路径的位置

#进程控制  
processManagement:
   fork: true                         #后台守护进程
   pidFilePath: <string>              #pid文件的位置，一般不用配置，可以去掉这行，自动生成到data中
    
#网络配置有关   
net:            
   bindIp: <ip>                       #监听地址
   port: <port>                       #端口号,默认不配置端口号，是27017
   
#安全验证有关配置      
security:
  authorization: enabled              #是否打开用户名密码验证
  
------------------以下是复制集与分片集群有关-------------
replication:
 oplogSizeMB: <NUM>
 replSetName: "<REPSETNAME>"
 secondaryIndexPrefetch: "all"
 
sharding:
   clusterRole: <string>
   archiveMovedChunks: <boolean>
      
---for mongos only
replication:
   localPingThresholdMs: <int>

sharding:
   configDB: <string>
```

yaml例子

```yaml
cat >  /mongodb/conf/mongo.conf <<EOF
systemLog:
   destination: file
   path: "/mongodb/log/mongodb.log"
   logAppend: true
storage:
   journal:
      enabled: true
   dbPath: "/mongodb/data/"
processManagement:
   fork: true
net:
   port: 27017
   bindIp: 10.0.0.51,127.0.0.1
EOF
```

重启MongoDB

```bash
mongod -f /mongodb/conf/mongo.conf --shutdown
mongod -f /mongodb/conf/mongo.conf   

#MongoDB的关闭方式
mongod -f mongo.conf --shutdown
```

MongoDB使用systemd管理

```bash
cat > /etc/systemd/system/mongod.service <<EOF
[Unit]
Description=mongodb 
After=network.target remote-fs.target nss-lookup.target
[Service]
User=mongod
Type=forking
ExecStart=/mongodb/bin/mongod --config /mongodb/conf/mongo.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/mongodb/bin/mongod --config /mongodb/conf/mongo.conf --shutdown
PrivateTmp=true  
[Install]
WantedBy=multi-user.target
EOF

[root@db01 ~]# systemctl restart mongod
[root@db01 ~]# systemctl stop mongod
[root@db01 ~]# systemctl start mongod
```

#### 3.MongoDB常用的基本操作

**3.1MongoDB默认存在的库**

test: 登录时默认存在的库,管理MongoDB有关的系统库

admin库: 系统预留库,MongoDB系统管理库

local库: 本地预留库,存储关键日志

config库: MongoDB配置信息库

```sql
show databases/show dbs
show tables/show collections
use admin 
db/select database()
```

**3.1 命令种类**

db 对象相关命令

```sql
db.[TAB][TAB]
db.help()
db.oldboy.[TAB][TAB]
db.oldboy.help()
```

rs 复制集有关(replication set):

```bash
rs.[TAB][TAB]
rs.help()
```

sh 分片集群(sharding cluster)

```bash
sh.[TAB][TAB]
sh.help()
```



