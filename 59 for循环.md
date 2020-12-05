## for循环

***1.什么是for循环***

脚本在执行任务的时候,总会遇到需要循环的时候,比如说我们需要 脚本

每五分钟执行一次ping的操作,除了计划任务我们还可以使用循环语句

***2.什么是for循环***

很多人把for循环叫做条件循环,因为for循环的次数和给予的条件是成正比的,也就是你给5个条件,那么他就循环五次.

**PS: for循环默认以空格为分割符**



### 语法

***1.for循环语法示例***

```bash
#for循环语法结构              
for 变量名 in [取值列表]
do
	循环体
done

#示例
for var in 1 2 3 
do
	echo "test is $var"
done 
```

***2.for循环中的负载值,可以使用引号或转义字符""来加以约束***

```bash
for var in "file2 file3" file4 "hello word"
do
	echo "the test is $var"
done 
```

***3.控制循环多少次***

```bash
for i in {1..5}
do
	echo $i
done 

for i in $(seq 5)
do
	echo $i
done 
```

***4.当条件中有特殊字符需要转义***

```bash
for var in "file2 file\'3 file\'4
do
	echo "the test is $var"
done 
```

***5. 从变量中取值***

```bash
list="file2 file3 file4 "
for var in $list
do
	echo "the test is $var"
done 
```

***6.读取文件循环***

```bash
#默认按空格循环
#以冒号作为分隔符循环	 			IFS=:
#以冒号,分号和双引号作为分割符	 IFS=:;"
#以换行符/回车作为分隔符		IFS=$'\n'

for var in $(cat /etc/hosts)
do
	echo $var
done 

IFS=$'\n'
for var in $(cat /etc/hosts)
do
	echo $var
done 
```

***7.c语言风格for***

```bash
for ((i=0;i<10;i++))
do
	commands
done
```



### 示例

需求1：判断主机存活状态，要求判断三次，如果三次失败则失败。

```bash
root@manager for]# cat for-12.sh 
#!/bin/bash

for i in {1..254}
do
    {
		ip=10.0.0.$i
		ping -W1 -c1 $ip &>/dev/null
		if [ $? -eq 0 ];then
			echo "$ip 存活" >> ok.txt
		else
			#如果判断第一次不存活,则在进行一次for循环,循环3次
			for j in {1..3}
			do
				ping -W1 -c1 $ip &>/dev/null
				if [ $? -eq 0 ];then
					echo "$ip 存活" >> ok.txt
				else
					echo "$ip 不存活" >> err.txt
				fi
			done
		fi
	}&
done
	wait
```

需求2：现在有一个ip.txt的文件，里面有很多IP地址。
	还有一个port.txt的文件，里面有很多端口号。
	现在希望对ip.txt的每个IP地址进行端口的探测,探测的端口号来源于port.txt文件中
	最后将开放的端口和IP保存到一个ok.txt文件。
ip.txt							port.txt
10.0.0.1						80
10.0.0.2						22
10.0.0.3						3306
10.0.0.4						23
10.0.0.5						443
10.0.0.6						9000
10.0.0.7						123
10.0.0.8						6379
10.0.0.9						10050
172.16.1.5						10051
192.168.10.1
172.16.1.6

```bash
[root@manager for]# cat for-13.sh 
#!/bin/bash

#遍历文件中的IP地址
for ip in $(cat ip.txt)
do
	#第二次循环,遍历文件中的端口号
	for port in $(cat port.txt)
	do
		#探测IP与端口的存活状态
		nc -z -w 1 $ip $port &>/dev/null
		if [ $? -eq 0 ];then
			echo "$ip $port is ok"
		fi
	done
done
```

需求3：循环批量创建用户，需要填入用户的数量、用户的前缀、用户的统一密码(使用read、case、for语句)****

```bash
[root@manager for]# cat useradd-passwd.sh 
#!/usr/bin/bash

read -p "请输入你需要创建的前缀: " qz
if [ -z $qz ];then
	echo "回车做什么  gdx...."
	exit
fi

read -p "请输入你需要创建的数量: " hz
if [[ ! $hz =~ ^[0-9]+$ ]];then
	echo "让你输数字,,,,emm"
	exit
fi

read -p "请输入所有用户统一的密码: " pw


read -p "你要创建的用户是 $qz , 个数是 $hz  密码是 $pw 你确定要创建吗? [ y | n ] "  Action
case $Action in
	y|yes|Y)
		for number in $(seq $hz)
		do
			username=$qz$number
			id $username &>/dev/null
			if [ $? -ne 0 ];then
				useradd $username
				echo "$pw" | passwd --stdin $username
				echo "$username $pw is create ok"			
			else
				#表示结束当前本次的循环,直接开始下一次循环
				continue
			fi
		done
		;;
	n|no|N)
			echo "Bey!"
			exit 
		;;
	*)
		echo "Gdx"
		exit
esac
```

需求15：编写一个上课随机点名脚本。

```bash
需求15：编写一个上课随机点名脚本。
[root@manager for]# cat for-20.sh 
#!/bin/bash

if [ -s name.txt ];then
	 User=$(sort --random-sort name.txt |awk 'NR==1')
	 echo "$User" 
	 
	 grep $User name.txt >> name1.txt
	 sed -i '/'$User'/d' name.txt

 else
 	cat name1.txt>name.txt
	rm -rf name1.txt
fi

```

