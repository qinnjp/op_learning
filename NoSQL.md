## Redis缓存

#### 1. NoSQL产品

RDBMS: MySQL, Oracle , MSSQL, PG

NoSQL: Redis, MongoDB, ES, HBASE

NewSQL: ----->分布式数据库架构(学习了MongoDB)

缓存产品介绍:

memcached  (大公司会做二次开发)

redis

Tair

#### 2. Redis功能介绍

数据类型丰富   `*****`

支持持久化     `*****`

多种内存分配及回收策略

支持事务

消息队列,消息订阅

支持高可用

支持分布式分片集群

缓存穿透\雪崩

Redis API

#### 2. 企业缓存产品介绍

Memcached: 

优点: 高性能读写,单一数据类型,支持客户端分布式集群,一致性hash多核结构,多线程读写性能高.

缺点: 无持久化,节点故障可能出现缓存穿透,分布式需要客户端实现,跨机房数据同步困难,架构扩容复杂度高.

Redis: 

优点: 高性能读写,多数据类型支持,数据持久化,高可用架构,支持自定义虚拟内存,支持分布式分片集群,单线程读写性能极高.

缺点: 多线程读写较Memcached慢

新浪,京东,直播类平台,网页游戏



Mencached与Redis在读写性能的对比:

Memcached 适合,多用户访问,每个用户少量的rw

redis       适合,少用户访问,每个用户大量rw



Tair:

优点: 高性能读写,支持三种存储引擎(ddb,rdb.ldb),支持高可用,支持分布式分片集群,支持了几乎所有淘宝业务的缓存.

缺点: 单机情况下,读写性能较其他两种产品较慢



#### 3. Redis使用场景介绍

Memcached: 多核的缓存服务,更加适合于多用户并发访问次数较少的应用场景.

Redis: 单核的缓存服务,单节点情况下,更加适合于少量用户,多次访问的应用场景.

Redis: 一般是单机多实例架构,配合redis集群实现.

#### 4. Redis安装部署

```bash
#下载：
wget http://download.redis.io/releases/redis-3.2.12.tar.gz

#解压：
mkdir /data
tar xzf redis-3.2.12.tar.gz
mv redis-3.2.12 redis

#安装：
yum -y install gcc automake autoconf libtool make
cd redis
make

#环境变量：
vim /etc/profile 
export PATH=/data/redis/src:$PATH
source /etc/profile 

#启动：
redis-server & 

#连接测试：
redis-cli 
127.0.0.1:6379> set num 10
OK
127.0.0.1:6379> get num
10
```

#### 5. Redis基本管理操作

**5.1 基础配置文件介绍**

```bash
mkdir /data/6379

cat > /data/6379/redis.conf<<EOF
daemonize yes
port 6379
logfile /data/6379/redis.log
dir /data/6379
dbfilename dump.rdb
EOF


redis-cli shutdown 
redis-server /data/6379/redis.conf 
netstat -lnp|grep 63

```

+++++++++++配置文件说明++++++++++++++

```bash
redis.conf
#是否后台运行：
daemonize yes
#默认端口：
port 6379
#日志文件位置
logfile /var/log/redis.log
#持久化文件存储位置
dir /data/6379
#RDB持久化数据文件:
dbfilename dump.rdb
++++++++++++++++++++++++++++++++++++++
redis-cli
127.0.0.1:6379> set name zhangsan 
OK
127.0.0.1:6379> get name
"zhangsan"
```

**5.2 Redis安全配置**

redis默认开启了保护模式，只允许本地回环地址登录并访问数据库。

Bind :指定IP进行监听.增加requirepass  {password}

```bash
vim /data/6379/redis.conf
bind 10.0.0.51  127.0.0.1
requirepass 123456

#验证
#方法一：
[root@db03 ~]# redis-cli -a 123456
127.0.0.1:6379> set name zhangsan 
OK
127.0.0.1:6379> exit
#方法二：
[root@db03 ~]# redis-cli
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> set a b
[root@db01 src]# redis-cli -a 123 -h 10.0.0.51 -p 6379
10.0.0.51:6379> set b 2
OK
```

**5.3 在线查看和修改配置**

```sql
CONFIG GET *
CONFIG GET requirepass
CONFIG GET r*
CONFIG SET requirepass 123
```

**5.4 redis持久化(内存数据保存到磁盘)**

RDB,AOF

RDB 持久化

可以在指定的时间间隔内生成数据集 时间点快照(point-in-time snapshot)

优点: 速度块,适合于用作备份,主从复制也是基于RDB持久化功能实现的

缺点: 会有数据缺失

rdb持久化核心配置参数: 

```bash
vim /data/6379/redis.conf
dir /data/6379
dbfilename dump.rdb
save 900 1
save 300 10
save 60 10000
```

配置分别表示: 

- 900秒(15分钟)内有1个更改
- 300秒(5分钟)内有10个更改
- 60秒内有10000个更改

说明: 

- 正常shutdown redis,自动触发save
- crash redis 异常宕机,不能自动触发save

AOF 持久化(append-only log file)

- 记录服务器执行所有写操作命令,并在服务器启动时,通过重新执行这些命令来还原数据集.

- AOF文件中的命令全部以Redis协议的格式来保存,新命令会被追加到文件的末尾

  优点: 可以最大程度保证数据不丢

  缺点: 日志记录量级比较大

AOF持久化配置

```bash
appendonly yes
appendfsync always
appendfsync everysec
appendfsync no
```

是否打开aof日志功能
每1个命令,都立即同步到aof 
每秒写1次
写入工作交给操作系统,由操作系统判断缓冲区大小,统一写入到aof.

```bash
vim /data/6379/redis.conf
appendonly yes
appendfsync everysec 
```



面试： 
redis 持久化方式有哪些？有什么区别？
rdb：基于快照的持久化，速度更快，一般用作备份，主从复制也是依赖于rdb持久化功能
aof：以追加的方式记录redis操作日志的文件。可以最大程度的保证redis数据安全，类似于mysql的binlog