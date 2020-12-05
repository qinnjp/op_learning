

```bash
双一标准之一
sync_binlog=1		#在事务提交的时候立即刷写binlog日志,保证二进制数据的完整性
```



## Percona Xtrabackup (物理)

#### 1. 安装

**1.1 安装依赖包：**

```bash
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

# 安装依赖
yum -y install perl perl-devel libaio libaio-devel perl-Time-HiRes perl-DBD-MySQL libev
```

**1.2 下载软件并安装**

```bash
wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.12/binary/redhat/7/x86_64/percona-xtrabackup-24-2.4.12-1.el7.x86_64.rpm

yum -y install percona-xtrabackup-*.rpm
```

#### 2. 备份命令介绍

xtrabackup

innobackupex

#### 3. 备份方式----物理备份

备份流程: 

- INNODB

  - 1 进行 checkpoint,将内存已经提交事务的脏页,刷新到磁盘idb文件,并且  记录LSN.
  - 2 拷贝InnoDB数据(idb,frm,ibdata1)文件到预定的备份路径

- NO-INNODB

  - 1 FLUSH TABLE WITH READ LOCK(FTWRL) Global READ LOCK
  - 2 拷贝非InnoDB表各种数据
  - 3 解锁

- REDO

  自动将备份期间的redo变化保存下来.并记录全新的LSN号码

恢复流程: 

- 1 prepare  准备过程

  - 1 redo 将变化前滚
  - 2 Undo将未提交的事务进行回滚

- 2 copy-back 恢复过程

  cp

  --copy-back

  授权



#### 4. 备份策略介绍

full + inc : 全备 + 增量

full + binlog : 全备 + 日志

#### 5. xbk备份工具的使用

**5.1 全备**

```bash
innobackupex  --user=root --password=123 /data/backup
```

**5.2 全备自定制目录名**

```bash
innobackupex  --user=root --password=123 --no-timestamp /data/backup/full_`date +%F`
```

**5.3 备份信息介绍**

- xtrabackup_binlog_info : 

  mysql-bin.000003	4432	411148c1-26bf-11ea-a420-000c298780da:1-32
  作用: 二进制日志截取的起点位置.

- xtrabackup_checkpoints :

  backup_type = full-backuped 
  from_lsn = 0
  to_lsn = 227111469
  last_lsn = 227111478
  compact = 0
  recover_binlog_info = 0

- xtrabackup_info

- xtrabackup_logfile

**5.4 增量备份**

- 1 模拟周一白天的数据变化: 

  ```sql
  create database xbk charset utf8mb4;
  use xbk;
  create table t1 (id int);
  create table t2 (id int);
  create table t3 (id int);
  insert into t1 values(1);
  commit;
  insert into t2 values(1);
  commit;
  insert into t3 values(1);
  commit;
  ```

- 2 模拟周一晚上的增量inc1

  ```bash
  innobackupex  --user=root --password=123456 --no-timestamp  --incremental  --incremental-basedir=/data/backup/full_2019-12-27    /data/backup/inc1
  ```

- 3 模拟周二白天数据变化

  ```sql
  use xbk;
  insert into t1 values(2);
  commit;
  insert into t2 values(2);
  commit;
  insert into t3 values(2);
  commit;
  ```

- 4 模拟周二晚上的增量inc2 

  ```bash
  innobackupex  --user=root --password=123456 --no-timestamp  --incremental  --incremental-basedir=/data/backup/inc1    /data/backup/inc2
  ```

- 5 模拟周三白天的数据变化

  ```sql
  use xbk;
  insert into t1 values(3);
  commit;
  insert into t2 values(3);
  commit;
  insert into t3 values(3);
  commit;
  ```

**5.5 故障恢复**

```bash
#模拟故障
pkill mysqld
\rm -rf /data/mysql/data_3306/*
```

恢复思路: 

- 1 找最近的全备

- 2 所有增量(inc1,inc2)合并全备(/data/backup/full_2019-12-27) 中

- 3 备份的prepare(应用redo和undo)
- 4 恢复备份cp,改权限
- 5 启动数据库
- 6 截取周二备份到周三故障点的binlog,进行恢复

恢复过程: 

- 1 增量合并并准备

  - 1 base_full 的 prepared --apply-log\

    ```bash
    innobackupex --apply-log --redo-only /data/backup/full_2019-12-27
    ```

    This option should be used when preparing the base full
    backup and when merging all incrementals except the last one. 
    各个备份的,lsn能够连续.

  - 2 将inc1合并到full中

    ```bash
    innobackupex --apply-log --redo-only   --incremental-dir=/data/backup/inc1 /data/backup/full_2019-12-27
    ```

  - 3 将inc2合并到full中

    ```bash
    innobackupex --apply-log  --incremental-dir=/data/backup/inc2 /data/backup/full_2019-12-27
    ```

  - 4 整体备份的prepare

    ```bash
    innobackupex --apply-log  /data/backup/full_2019-12-27
    ```

- 2 恢复备份

  ```bash
  cp -a * /data/mysql/data_3306/
  chown -R mysql.mysql /data
  ```

- 3 启动数据库

  ```bash
  /etc/init.d/mysqld start
  ```

- 4 截取二进制日志

  ```bash
  [root@db01 inc2]# cat /data/backup/inc2/xtrabackup_binlog_info 
  mysql-bin.000003	6593	411148c1-26bf-11ea-a420-000c298780da:1-42
  [root@db01 inc2]# mysqlbinlog --skip-gtids --include-gtids='411148c1-26bf-11ea-a420-000c298780da:43-45' /data/mysql/binlog_3306/mysql-bin.000003 >/data/backup/bin.sql
  
  ```

- 5 恢复binlog

  ```SQL
  mysql> set sql_log_bin=0;
  mysql> source /data/backup/bin.sql
  mysql> set sql_log_bin=1;
  ```

- 6 总备份2T,误删除1G一张核心表

  思路: 表空间迁移













面试题: xbk 在innodb表备份恢复的流程
0、xbk备份执行的瞬间，立即触发ckpt,已提交的数据脏页,从内存刷写到磁盘,并记录此时的LSN号
1、备份时，拷贝磁盘数据页，并且记录备份过程中产生的redo和undo一起拷贝走,也就是checkpoint LSN之后的日志
2、在恢复之前，模拟Innodb“自动故障恢复"的过程，将redo (前滚)与undo (回滚)进行应用
3、恢复过程是cp备份到原来数据目录下















