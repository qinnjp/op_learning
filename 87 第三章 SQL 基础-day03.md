## 第三章 SQL 基础

#### 1. SQL介绍

**1.1 简介**

结构化查询语言. 
**1.2 SQL标准** 

SQL89   SQL92  SQL99  SQL03  SQL05  

**1.3 SQL_MODE**

除数为零
日期
mysql> select @@sql_mode;

**1.4 SQL 类型**

DDL : 数据定义语言 : 库名,库属性,表名,表属性,列(列名,列属性)
DCL : 数据控制语言 : 权限
DML : 数据操作语言 : 数据行
DQL : 数据查询语言 : 数据行

**1.5 SQL功能**

管理,操作数据库对象: 
	库:  库名,库属性
	表:  表名,表属性,列(列名,列属性),数据行

#### 2. MySQL规范性存储限制 

**2.1 字符集Charset**

utf8     :  最大字节长度3个.
utf8mb4  :  最大字节长度4个. 可以存储emoji表情字符.
mysql> show charset;

**2.2 排序规则 (校对规则)**

show  collation;
默认是大小写不敏感.
utf8mb4_general_ci 
utf8mb4_bin

**2.3 数据类型**

*2.3.1 数字类型*

```bash
tinyint   1字节长度数字  ===> 11111111  ===> 0-2^8-1 ===> -2^7-2^7-1 (3位)

int       4字节长度      ====> 0-2^32-1 ====> -2^31 - 2^31-1  (10位数)

bigint    8字节长度		 ====> 0-2^64-1 ====> -2^63 - 2^63-1  (20位数)
```

*2.3.2 字符串*

char(10)  : 定长类型,最多10个字符,占用存储空间一定.最多存储255个字符.
varchar(10): 
	变长类型,最多10个字符,按需分配存储空间.
	需要额外1个字符或2个字符存储字符长度

因素: 变长的字符串列,90%几率都是varchar
具体原因是什么?
节省空间,还有没有别的原因?
遗留的问题..

枚举类型: 

enum('m','f')
	        1   2 
  	  

*2.3.3 时间类型*

DATETIME 
范围为从 1000-01-01 00:00:00.000000 至 9999-12-31 23:59:59.999999。

TIMESTAMP 
1970-01-01 00:00:00.000000 至 2038-01-19 03:14:07.999999。
timestamp会受到时区的影响

*2.3.4 二进制类型*
2.3.5 JSON(8.0)

### 3.约束

**3.1 `PrimaryKey(PK)`:主键**

特点: 唯一+非空,一张表中只能有一个主键约束.一般是一个数字列,最好是无意义的

**3.2 `NOT NULL` 非空**

特点: 不能为空,我们建议业务关键列(索引列),尽量设置成非空.

**3.3 `UNIQUE` 唯一约束**

特点: 不能有重复值,一般像手机号,身份证号,qq,邮箱......

**3.4 `unsigned` 数字列无符号**

特点: 必须要加载数字列后,表示数字无负数,一般适用于年龄...

#### 2. 其他属性

**2.1 `AUTO_INCREMENT` 自增长**

特点: 适用于ID主键列.

**2.2 `DEFAULT` 默认值**

特点: 使用在NOT NULL 列中,不填写值时,自动生成默认值。
**2.3 `COMMENT` 注释****

特点: 建议每个列都有一个注释.
10张表: -----> 每个表每个列加注释.

#### 4. DCL 数据控制语言(略) 

grant 
revoke

注:见第二章

#### 4. DDL应用

**4.1 库**

- 增

```bash
CREATE DATABASE oldguo CHARSET utf8mb4 COLLATE utf8mb4_bin;
```

- 删

```bash
DROP DATABASE oldguo;
```

- 改

```bash
ALTER DATABASE oldguo CHARSET utf8mb4 COLLATE utf8mb4_bin;
```

- 差

```bash
mysql> show databases;
mysql> show create database oldboy;
```

*4.1.1 规范*

==(1) 库名要和业务有关==
==(2) 库名不能有大写字母,可以有小写字符,数字,特殊符号.==
==为什么?==
==CREATE DATABASE  OLDGUO  CHARSET utf8mb4 COLLATE utf8mb4_bin;==
==(3) 库名不能是数字开头==
==(4) 库名不能是预留字符==
==(5) 库名不要超过18个字符.==
==(6) 必须要设置字符集.尽量是utf8mb4.==
==(7) 收回所有用户的DROP权限.==

**4.2 表定义**
*4.2.1 添加表*

```sql
# 学生表:student
drop table student;
CREATE TABLE `student` (
  `xid` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '学号',
  `xname` varchar(64) COLLATE utf8mb4_bin NOT NULL COMMENT '姓名',
  `xage` tinyint(3) unsigned NOT NULL DEFAULT '99' COMMENT '年龄',
  `xsex` char(1) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '性别',
  `xtel` char(14) COLLATE utf8mb4_bin NOT NULL COMMENT '手机号',
  `xcard` char(18) COLLATE utf8mb4_bin NOT NULL COMMENT '身份证号',
  `xaddr` enum('北京市','上海市','深圳市','山东省','甘肃省','河北省','山西省','河南省','辽宁省','吉林省','黑龙江省','内蒙古自治区','新疆维吾尔自治区','四川省','陕西省','江苏省','福建省','湖北省','广东省','广西省') COLLATE utf8mb4_bin NOT NULL DEFAULT '北京市' COMMENT '地区',
  `xdate` datetime DEFAULT NULL COMMENT '入学时间',
  PRIMARY KEY (`xid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin ;  

INSERT INTO student(xid,   xname , xage    ,xsex , xtel , xcard  ,xaddr ,xdate)
VALUES
(1 ,    '张三'   , 11  ,   'm'  ,   '110'  ,  '660',   '北京市',            '2019-01-01'),
(2 ,    '马六'   , 13  ,   'm'  ,   '111'  ,  '661',   '上海市',            '2019-01-01'),
(3 ,    '李四'   , 14  ,   'm'  ,   '112'  ,  '662',   '北京市',            '2019-01-01'),
(4 ,    '王五'   , 17  ,   'm'  ,   '113'  ,  '663',   '山东省',            '2019-01-01'),
(5 ,    '铁锤'   , 18  ,   'f'  ,   '114'  ,  '664',   '河南省',            '2019-01-01'),
(6 ,    '钢蛋'   , 13  ,   'f'  ,   '115'  ,  '665',   '河北省',            '2019-01-01'),
(7 ,    '孙悟空' , 19  ,   'm'  ,   '116'  ,  '666',   '山西省',            '2019-01-01'),
(8 ,    '猪八戒' , 21  ,   'm'  ,   '117'  ,  '667',   '河北省',            '2019-01-01'),
(9 ,    '唐僧'    ,23   ,  'm'   ,  '118'   , '668' ,  '吉林省',            '2019-01-01'),
(10,    '沙僧'    ,31   ,  'm'   ,  '120'   , '669' ,  '辽宁省',            '2019-01-01'),
(11,    '白龙马'  ,26   ,  'm'   ,  '119'   , '670' ,  '广西省',           '2019-01-01') ,
(12,    '牛魔王'  ,19   ,  'm'   ,  '121'   , '671' ,  '四川省',            '2019-01-01'),
(13,    '张无忌'  ,20   ,  'm'   ,  '122'   , '672' ,  '福建省',            '2019-01-01'),
(14,    '赵敏'    ,28   ,  'f'   ,  '123'   , '673' ,  '广东省',            '2019-01-01'),
(15,    '郭靖'    ,29   ,  'm'   ,  '124'   , '674' ,  '甘肃省',            '2019-01-01'),
(16,    '黄蓉'    ,17   ,  'f'   ,  '125'   , '675' ,  '深圳市',            '2019-01-01'),
(17,    '小龙女'  ,22   ,  'f'   ,  '126'   , '766' ,  '黑龙江省',     	   '2019-01-01'),
(18,    '杨过'    ,33   ,  'm'   ,  '127'   , '777' ,  '新疆维吾尔自治区',  '2019-01-01'),
(19,    '欧阳峰'  ,25   ,  'm'   ,  '128'   , '888' ,  '内蒙古自治区',      '2019-01-01'),
(20,    '小沈阳'  ,23   ,  'm'   ,  '129'   , '999' ,  '陕西省',            '2019-01-01');

SELECT * FROM student;

# 课程表:course
drop table course;
CREATE TABLE course (
cid  INT NOT NULL PRIMARY KEY COMMENT '课程编号',
cname VARCHAR(64) NOT NULL COMMENT '课程名称',
tid   CHAR(5) NOT NULL COMMENT '讲师名',
cprice INT NOT NULL COMMENT '课程价格'
)ENGINE=INNODB CHARSET=utf8mb4;

insert into course(cid,  cname , tid    ,cprice )
values
(1001, 'linux'  ,'t0001' ,19800),  
(1002, 'python' ,'t0002' ,21800),  
(1003, 'golang' ,'t0003' ,16000),  
(1004, 'DBA'    ,'t0004' ,15000),  
(1005, 'safe'   ,'t0005' ,17800);  



# 教师表 : teacher
CREATE TABLE teacher (
tid CHAR(5) NOT NULL PRIMARY KEY COMMENT '教师编号',
tname VARCHAR(64) NOT NULL  COMMENT '教师姓名',
tage TINYINT NOT NULL  DEFAULT 99 COMMENT '教师年龄',
tsex CHAR(1) NOT NULL DEFAULT 'm' COMMENT '教师性别',
tyear TINYINT NOT NULL  DEFAULT 3 COMMENT '工作年限',
txl  VARCHAR(64) NOT NULL DEFAULT '本科' COMMENT '学历',
tstar TINYINT NOT NULL DEFAULT 5 COMMENT '级别:1-10'
)ENGINE=INNODB CHARSET=utf8mb4;


# 成绩表 : score
DROP TABLE score;
CREATE TABLE score (
xid INT NOT NULL COMMENT '学生编号',
cid INT  NOT NULL COMMENT '课程编号',
score INT NOT NULL DEFAULT 0 COMMENT '课程分数',
quekao TINYINT NOT NULL DEFAULT 0 COMMENT '是否缺考:1缺考,0未缺考'
)ENGINE=INNODB CHARSET=utf8mb4;

INSERT INTO score(xid  ,   cid  ,  score ,quekao) 
VALUES
(1        ,1001   ,80     ,0),
(1        ,1002   ,70     ,0),
(2        ,1001   ,0      ,1),
(2        ,1003   ,90     ,0),
(3        ,1004   ,80     ,0),
(4        ,1004   ,100    ,0), 
(5        ,1005   ,60     ,0), 
(4        ,1005   ,30     ,0), 
(5        ,1002   ,60     ,0), 
(6        ,1002   ,45     ,0), 
(7        ,1003   ,67     ,0), 
(7        ,1004   ,98     ,0), 
(8        ,1004   ,76     ,0), 
(9        ,1001   ,80     ,0),  
(10       ,1002   ,99     ,0), 
(11       ,1003   ,40     ,0),
(11       ,1004   ,50     ,0),
(12       ,1005   ,0      ,1),
(12       ,1003   ,90     ,0),
(13       ,1001   ,30     ,0),
(14       ,1002   ,100    ,0), 
(15       ,1003   ,60     ,0), 
(14       ,1004   ,30     ,0), 
(15       ,1005   ,60     ,0), 
(16       ,1002   ,45     ,0), 
(17       ,1003   ,67     ,0), 
(17       ,1004   ,98     ,0), 
(18       ,1004   ,76     ,0), 
(19       ,1005   ,75     ,0),  
(20       ,1002   ,68     ,0); 
```

`建表规范:   *****`

1.  ==表名:  不能大写字母,和业务有关,不能数字开头,长度控制在18字符以内,不能和关键字同名.==
2.  ==要设置存储引擎类型:INNODB,要设置字符集==
3.  ==列名要有意义==
4.  ==合适的,完整的,简短的数据类型.(会影响到索引的性能)==
5.  ==每个表要有主键,实在不知道怎么设置,也要找一个无关的自增长列设置为主键.==
6.  ==尽量每个列都有not null (特别是将来要做为索引的列)==
7.  ==每个列要有注释信息==

mysql 开发规范

https://www.cnblogs.com/jun1019/p/9353862.html

小扩展:

```sql
克隆表结构
mysql> create table teacher_bak like teacher;
```

*4.2.2 删(危险! 谨慎操作!)*

```sql
mysql> drop table teacher_bak;
mysql> truncate table teacher;
```

面试: 请你说明 drop table   truncate delete table 区别 ?

drop table : 表结构+数据(物理性删除)

truncate table : 数据(清空数据页)

delete from table: 清空数据行(逐行删除)



*4.2.3 改* 
(1) 添加列 

```sql
DESC xuesheng;
ALTER TABLE xuesheng ADD xqq  BIGINT  NOT NULL UNIQUE COMMENT 'qq号';
ALTER TABLE xuesheng ADD wechat  BIGINT  NOT NULL UNIQUE COMMENT '微信号' AFTER xtel;
ALTER TABLE xuesheng ADD mail  BIGINT  NOT NULL UNIQUE COMMENT '邮箱' FIRST'
```

(2) 删除列

```sql
ALTER TABLE xuesheng DROP mail;
ALTER TABLE xuesheng DROP xqq;
ALTER TABLE xuesheng DROP wechat;
ALTER TABLE xuesheng DROP mail;
```

(3) 修改
1. 修改表名:

```sql
ALTER TABLE xuesheng RENAME TO student;
```

面试题: 
上亿行的数据规划:

	1. 按月归档表
	2. 没用的历史表进行挪走或删除
pt-archiver  自己扩展 

2. 修改某一列的属性信息(优先选择)

```sql
DESC student;
ALTER TABLE student MODIFY xname VARCHAR(128) NOT NULL COMMENT '姓名';
```

3. 修改列名和属性

```sql
ALTER TABLE student CHANGE xsex xgender CHAR(2) NOT NULL DEFAULT 'm' COMMENT '性别';
```

注意: 
	执行alter语句,都是需要进行锁表操作的,此时只能发生查询操作,不能做修改操作.
	我们建议,alter语句,尽量在业务不繁忙期间发生.如果非得线上操作,建议使用pt-osc工具进行.

*4.2.4 查*

```sql
show tables ;
desc teacher;
show create table xx;
```

#### 5. DML  数据操作语言
**5.1 insert 插入数据**

```sql
insert into 
course(cid,  cname , tid    ,cprice )
values
(1001, 'linux'  ,'t0001' ,19800),  
(1002, 'python' ,'t0002' ,21800),  
(1003, 'golang' ,'t0003' ,16000),  
(1004, 'DBA'    ,'t0004' ,15000),  
(1005, 'safe'   ,'t0005' ,17800);  
```

**5.2 update** 

```sql
SELECT * FROM student;
UPDATE student SET xname='王钢蛋' WHERE xid=6;
UPDATE student SET xname='李铁锤' WHERE xid=5;
```

**5.3 delete**

```sql
INSERT INTO student VALUES(21,'王二麻子',22,'f','921','345','上海市','2020-01-01');
DELETE FROM student WHERE xid=21;
```

- 扩展: 

**伪删除**

(1) 添加状态列 is_del  (1代表删除,0代表有效)

```sql
ALTER TABLE student ADD is_del TINYINT NOT NULL DEFAULT 0 COMMENT '1代表删除,0代表有效';
SELECT * FROM student;
```

(2) delete   --->  update 

```sql
原语句:

delete from student where xid=20;

改为 :
update student set is_del=1 where xid=20;
```

(3) 更改业务查询方法

```sql
原语句: 
SELECT * FROM student;
改为: 
SELECT * FROM student where is_del=0;
```

#### 注:

1.create table like 只能复制表结构,有什么命令可以连数据一起复制?

- 方法一: 

  ```sql
  create table stu select * from student;
  #主键等特性没有被复制.
  ```

- 方法二: 

  ```sql
  create table stu1 like student;
  insert into stu1 select * from student;
  ```

2.主键是干什么用的

- 1.约束; unique not null
- 2.聚簇索引索引: 组织和存储数据.
- 3.加速查询

3.update和delete范围操作

```sql
update t1 set name='王二麻子' where name link "王二%";
delete from t1 where name like '王二%';
delete from t1 where xid>1 and xid<5;
delete from t1 where id between 1 and 5;
```



#### 6.DQL 数据查询语言.  ---oldguo

#### 1.select

1.1 select 单独使用

 (1) 查询数据库的参数.

```sql
SELECT @@port;				-- 查看端口
SELECT @@datadir;     		-- 查看数据目录地址
SELECT @@basedir;			-- 查看数据库安装路径
SELECT @@innodb_flush_log_at_trx_commit; 
SHOW VARIABLES;				-- 打印所有数据库参数
SHOW VARIABLES  LIKE '%trx%';
```

 (2) 调用内置函数.

```sql
USE oldguo
SELECT DATABASE();		-- 查看当前所在的库
SELECT NOW();			-- 查看当前的时间 
SELECT CONCAT(USER,"@",HOST) FROM mysql.user; -- 查询用户并显示为user@host类型
SELECT GROUP_CONCAT(xid) FROM student;  -- 列转行
SELECT SUM(xid) FROM student;
```

 (3) 简易计算器

```sql
SELECT 4*5;
```

**1.2 select 配合其他子句使用** 

*1.2.1 子句列表介绍*

```sql
FROM     -- 查询对象(表,视图)
WHERE    -- 过滤子句(grep)
GROUP BY -- 分组子句(统计分析类)
HAVING   -- 后过滤子句
ORDER BY -- 排序子句
LIMIT    -- 限制子句(分页子句)
```

*1.2.2 配合FROM应用*

```sql
--  world 模板库介绍 
--- 英文单词介绍
--- city  -- 城市
--- id    -- 序号ID主键
--- NAME  -- 城市名
--- Countrycode -- 国家代码(CHN,USA,JPN)
--- District    -- 省,州
--- Population  -- 城市人口数
```

例子

```sql
-- 1.查询表中所有数据(cat)
SELECT * FROM city;
-- 2. 查询name和population信息 (awk取列)
SELECT NAME,population FROM city;
```

*1.2.3 select+ from + where(grep)使用*

```sql
-- where 配合等值查询
-- 例子:

-- 1. 查询中国所有的城市信息
SELECT * FROM city
WHERE countrycode='CHN';

-- 2. 查询ID为100的城市信息
SELECT * FROM city
WHERE id=100;

-- 3. 查询 中国河北省的城市信息
SELECT * FROM city
WHERE countrycode='CHN' AND district='hebei'  ;

-- 4. 查询 中国或者美国的城市
SELECT * FROM city
WHERE countrycode='CHN' OR countrycode='USA';
或者: 
SELECT * FROM city
WHERE countrycode IN ('CHN','USA');
或者: 
SELECT * FROM city
WHERE countrycode='CHN' 
UNION ALL 
SELECT * FROM city
WHERE countrycode='USA' ;
-- 说明:一般情况,我们会将 IN 或者 OR 语句 改写成 UNION ALL,来提高性能.
UNION    		-- 去重复
UNION ALL 		-- 不去重复
-------------------------------------------------------
-- where 配合范围查询

-- 例子 : 
-- 1. 查询人口数量小于100人的城市
SELECT *  FROM city
WHERE population<100;

-- 2. 查询人口数量100w-200w之间的
SELECT * FROM city 
WHERE population>=1000000 AND population<=2000000 ;
或者:
SELECT * FROM city 
WHERE population BETWEEN 1000000 AND 2000000 ;

-- 3. 查询国家代号是CH开头的城市信息
SELECT * FROM city 
WHERE countrycode LIKE 'CH%';
```

*1.2.4  group by 分组子句+聚合函数应用*

```sql
-- 聚合函数?
COUNT()  -- 计数
SUM()    -- 求和
AVG()    -- 求平均值
MAX()    -- 求最大值
MIN()    -- 最小值
GROUP_CONCAT() -- 聚合列值 

-- 结果集显示特点: 必须是1v1,不能是1vN
-- 例子 :
-- 1. 统计一下每个国家的人口总数
SELECT countrycode,SUM(population) 
FROM  city
GROUP BY countrycode;

-- 2. 统计中国每个省的人口总数
SELECT  district,SUM(population)  FROM  city
WHERE countrycode='CHN'
GROUP BY district;

-- 3. 统计下中国每个省的城市个数及城市名.
SELECT  district,COUNT(NAME),GROUP_CONCAT(NAME)  FROM  city
WHERE countrycode='CHN'
GROUP BY district;

-- 4. 统计每个国家城市个数
SELECT  countrycode ,COUNT(NAME) FROM  city
GROUP BY countrycode;
```

*1.2.5 having 后判断* 

```sql
-- 1. 统计中国每个省的人口总数,只显示总人口数大于500w的省信息.
SELECT  district,SUM(population)  FROM  city
WHERE countrycode='CHN'
GROUP BY district
HAVING SUM(population)  >=5000000;
```

*1.2.6  order by 排序子句*

```sql
-- 例子:
-- 1. 查询中国所有城市信息,人口数从大到小排序输出.
SELECT * FROM city
WHERE countrycode='CHN'
ORDER BY population DESC ;

-- 2. 查询中国所有城市信息,按城市名排序.
SELECT * FROM city
WHERE countrycode='CHN'
ORDER BY NAME;

-- 3. 查询中国所有省的总人口,并按总人口数从大到小排序输出.

SELECT  district,SUM(population)  FROM  city
WHERE countrycode='CHN'
GROUP BY district
ORDER BY SUM(population) DESC;
```

*1.2.7 limit 分页限制子句*

```sql
-- 查询中国所有省的总人口,并按总人口数从大到小排序输出.

SELECT  district,SUM(population)  FROM  city
WHERE countrycode='CHN'
GROUP BY district
ORDER BY SUM(population) DESC
LIMIT 5 OFFSET 1;
-- 跳过1行显示5行

SELECT  district,SUM(population)  FROM  city
WHERE countrycode='CHN'
GROUP BY district
ORDER BY SUM(population) DESC
LIMIT 10;


SELECT  district,SUM(population)  FROM  city
WHERE countrycode='CHN'
GROUP BY district
ORDER BY SUM(population) DESC
LIMIT 1,5;
-- 跳过1行显示5行

-- 注意: LIMIT 谨慎使用, 500w+的表.
-- LIMIT 5000000,100
-- 一般会改为明确查找范围
```

*1.2.8 distinct：去重复*

```sql
SELECT countrycode FROM city ;
SELECT DISTINCT(countrycode) FROM city  ;
```







































