## shell初识
1. 什么是Shell
	- shell一个命令解释器,   将人类输入高级语言, 通过 Shell程序转换为二进制 .
	
	- shell分为两种使用方式:
		交互:		登录 执行命令 退出
		非交互:		执行某个文件, 文件中都是一推命令, 整个文件从上往下依次执行.

2. 什么是Shell脚本
	1) 将系统命令堆积在一起，顺序执行(简称: 系统命令堆积)
	2) 特定的格式 + 特定的语法 + 系统的命令 = 文件。  Shell脚本
					if for ...

3. Shell脚本能做什么

	- 标准:
		1.安装方式一致
		2.安装路径一致
		3.目录结构一致
		4.日志格式一致
		5.脚本路径一致
	- 能将平时操作脚本化，流程化，自动化     ITIL 
		ppt   人  流程   技术/工具
	备份
	监控
	自动化上线
	约束标准

4. shell脚本需要的预备知识
5. shell脚本如何才能学好
	思路
	练习
	分享    (   )

6. Shell脚本编写规范、执行方式。

	执行方式分为两种:
		1.加执行权限  				./script_filename.sh
		2.通过bash直接翻译   bash  script_filename.sh


 - 脚本中#!/usr/bin/bash  加与不加区别在哪?
	1.如果你明确清楚这是一个什么类型的脚本,直接调用对应的解释器执行,没有影响?
	2.如果你不清楚是什么类型的脚本, 直接使用./执行,那么会读取该脚本的第一行
		如果第一行是#!/usr/bin/bash 或者 没有写该行,那么都将使用默认的bash翻译
		问题: 如果我是一个python脚本,没有写开头,那么执行一定会报错
		因为默认查找的是bash解释器,而我的文件需要用python解释器来翻译.

	添加命令解释器的几种方式:
		#!/usr/bin/bash
		#!/usr/bin/sh
		#!/usr/bin/env bash
		#!/usr/bin/env python

## 变量
1.什么是变量
变量是shell中传递数据的一种方法。
简单理解: 就是用一个固定的字符串去表示不固定的值，便于后续引用。

2.定义变量规范
	1.大写开头,后面小写或数字都ok
	2.变量具体一定的含义
	3.注意  变量的写法 仅支持 a=1         不支持  a = 1
	
	
3.自定义变量
var="hello world"

echo $var

echo ${var}_log

PS: 单引号和双引号的区别

​	单引号: 所见即所得

​	双引号: 解析所包含的变量



***2.系统环境变量示例,在当前shell和子shell有效***

1. 使用系统已定义好的环境变量

```bash
#查看系统所有的环境变量
[root@ssh02 variables]# set | less
```

2. 人为定义环境变量: export变量,将自定义变量转换成环境变量

```bash
#####
$* 和 $@ 的区别?
可以看到不加引号时,二者都是返回传入的参数,但加了引号后,此时$*把参数作为一个字符串整体(单字符串)返回,$@把每个参数作为一个字符串返回
```



***3.预先定义的变量参数示例,系统内置变量的使用方法***

```bash
#脚本内容
[root@ssh02 variables]# cat test.sh 
#!/bin/bash

echo "当前shell脚本的文件名: $0"
echo "第1个shell脚本位置参数: $1"
echo "第2个shell脚本位置参数: $2"
echo "第3个shell脚本位置参数: $3"
echo "所有传递的位置参数是: $*"
echo "所有传递的位置参数是: $@"
echo "总共传递的参数个数是: $#"
echo "当前程序运行的PID第: $$"
echo "上一个命令执行的返回结果: $?"

#执行结果
[root@ssh02 variables]# sh test.sh 11 22 33 44 
当前shell脚本的文件名: test.sh
第1个shell脚本位置参数: 11
第2个shell脚本位置参数: 22
第3个shell脚本位置参数: 33
所有传递的位置参数是: 11 22 33 44
所有传递的位置参数是: 11 22 33 44
总共传递的参数个数是: 4
当前程序运行的PID第: 9104
上一个命令执行的返回结果: 0
```

需求1：通过位置传参方式, 创建 Linux 系统账户及密码，执行 var1.sh username password

```bash
[root@ssh02 scripts]# cat var01.sh 
#!/bin/bash

useradd $1
echo "$2" | passwd --stdin $1
```

需求2：通过位置传参方式,  Linux 系统账户及密码，执行 var1.sh username password，控制最多传递两个参数。

```bash
[root@ssh02 scripts]# cat var02.sh 
#!/bin/bash

if [ $# -ne 2 ];then
	echo "USAGE: $0 请传递两个参数 [ username | password ]"
fi
useradd $1
echo "$2" | passwd --stdin $1

```

## 变量赋值

除了自定义变量,以及系统内置变量,还可以使用read命令通过交互式方式传递变量

| read选项 | 选项含义     |
| -------- | ------------ |
| -p       | 打印信息     |
| -t       | 限定时间     |
| -s       | 不回显       |
| -n       | 指定字符个数 |

1.read示例语法

需求1：使用read模拟Linux登陆页面。
	1.先实现, 无论多low
	2.在进行改进

```bash
[root@manager variables]# cat var08.sh 
#!/bin/bash


echo "$(hostnamectl |awk -F ":" '/Operating System:/ {print $2}')"
echo "Kernel $(uname -r) on an $(uname -m)"

read -p "$(hostname) Login: " acc
read -s -t50 -p "Password: " pw
echo ""
echo "你输入的用户名: $acc	你输入的密码是: $pw"
```





需求2：使用 read编写一个备份脚本，需要用户传递2个参数，源和目标。

 ```bash
[root@manager variables]# cat var09.sh 
#!/bin/bash


echo "----------请按照如下提示输入---------------"

read -p "请输入你要备份的源文件或者源目录: " Source
read -p "请输入你要备份的目录位置是哪里: " Dest

echo -e "\t你要备份的源是 $Source
       你要备份的目标是: $Dest"
read -p "你确定要备份吗? [ y | n ] " Action

if [ $Action == "y" ];then
	cp -rpv $Source $Dest
fi
 ```



需求3：使用read编写一个探测主机存活脚本，需要用户传递测试的IP地址。
	1.提示用户输入IP地址
	2.对用户输入的IP地址进行探测是否存活
	3.判断探测结果是否成功
		成功则输出成功的结果
		失败则输出失败的结果
		

```bash
[root@manager variables]# cat var10.sh 
#!/bin/bash

read -p "请输入需要探测的IP地址: " ip

ping -W1 -c1 "$ip" &>/dev/null
if [ $? -eq 0 ];then
	echo "$ip" is ok...
else
	echo "$ip" is err...
fi
```





需求4：使用read编写一个修改系统主机名称脚本。
	1.打印当前的主机名称
	2.提示用户输入新的主机名称
	3.询问用户是否修改?
	4.确定修改,执行修改命令
	5.不确定修改,退出脚本

```bash
[root@manager variables]# cat var11.sh 
#!/bin/bash

Hostname=$(hostname)
echo "当前的主机名称是: ${Hostname}"
read -p "请输入新的主机名称: " new_host
read -p "你确定要将 ${Hostname} 变更为 ${new_host} 吗? [ y | n ] " Action

if [ $Action == "y" ];then
	hostname ${new_host}
	hostnamectl set-hostname ${new_host}
	echo "你的主机名称已修改为  ${new_host} "
fi
```

## shell变量替换

1.什么是变量替换?

简单来说,就是再不改变原有变量的情况下,对变量进行替换.

2.为什么要用变量替换?

比如: 我们需要将变量的值进行整数对比,但变量的值是一个小数.我们就可以使用变量替换的方式,将小数位进行删除,然后进行比对.

3.变量替换的几种方式.

| 变量                       | 说明                              |
| -------------------------- | --------------------------------- |
| ${变量#匹配规则}           | 从头开始匹配,最短删除             |
| ${变量##匹配规则}          | 从头开始匹配,最长删除             |
| ${变量%匹配规则}           | 从尾开始匹配,最短删除             |
| ${变量%%匹配规则}          | 从尾开始匹配,最长删除             |
| ${变量/旧字符串/新字符串}  | 替换变量内的旧字符串,只替换第一个 |
| ${变量//旧字符串/新字符串} | 替换变量内的旧字符串,全部替换     |

需求2：变量string="Bigdata process is Hadoop, Hadoop is open source project"，执行脚本后，打印输出string变量，并给出用户以下选项：
1)、打印string长度
2)、删除字符串中所有的Hadoop
3)、替换第一个Hadoop为Linux
4)、替换全部Hadoop为Linux
用户输入数字1|2|3|4，可以执行对应项的功能

```bash
[root@manager variables]# cat var12.sh 
#!/bin/bash
# Author:      Oldux.com QQ: 552408925
# Date：       2019-10-28
# FileName：   var12.sh
# URL:         https://www.xuliangwei.com
# Description： 
string="Bigdata process is Hadoop, Hadoop is open source project"

echo $string
echo "1)、打印string长度"
echo "2)、删除字符串中所有的Hadoop"
echo "3)、替换第一个Hadoop为Linux"
echo "4)、替换全部Hadoop为Linux"
read -p "请输入对应的选项 [ 1 | 2 | 3 | 4 | q ] " Action

if [ $Action -eq 1 ];then
	echo "他的长度是: ${#string}"
fi

if [ $Action -eq 2 ];then
        echo "${string//Hadoop/}"
fi

if [ $Action -eq 3 ];then
	echo ${string/Hadoop/Linux}
fi

if [ $Action -eq 4 ];then
	echo ${string//Hadoop/Linux}
fi
```

需求3：查看内存/当前使用状态，如果使用率超过80%则报警发邮件，思路如下:
	1.当前内存使用百分比是多少
	2.进行判断比对 
		如果大于80%  则触发邮件
		否则,over
		已使用的内存  /  总内存  * 100 = 使用的百分比

```bash
mem_use=$(free -m | awk '/^Mem/ {print $3/$2*100}')
if [ ${mem_use%.*}  -ge 10 ];then
	echo "你的内存已经超过了80%  目前的内存使用状态是 ${mem_use}%"
fi
```

需求1：根据系统时间，打印今年和明年时间。

需求2：根据系统时间获取今年还剩下多少星期，已经过了多少星期。思路如下:
	date +%j  已经过了多少天

一年有365天   已经过了 301 = 还剩下   365-301 = 64  / 7 = 还剩下多少周
			  已经过了 301 天 / 7 = 已经过了多少周

需求3：完成一个计算器功能: 传入2个值，然后对传入的值进行 加 减 乘 除，思路如下:
	1.使用read让用户传值:     $1 $2
	2.对传入的值进行运算:
	3.输出对应的结果.

```bash
[root@ssh02 variables]# cat var08.sh 
#!/bin/bash

echo "今年是$(date +%Y)"
echo "明年是$(($(date +%Y)+1))"

echo "今年已经过了$(($(date +%j)/7)),还剩$(($(( 365 - $(date +%j)))/7))周"

read -p "请输入第一个数字：" num1
read -p "请输入第二个数字：" num2
read -p "请输入运算符" Action

if [ $Action = "+" ];then
echo "两个数的和为: $(($num1+$num2))"
fi

if [ $Action = "-" ];then
echo "两个数的差为: $(($num1-$num2))"
fi

if [ $Action = "x" ];then
echo "两个数的乘积为: $(($num1 * $num2))"
fi

if [ $Action = "/" ];then
echo "两个数的商为: $(($num1/$num2))"
fi

```

