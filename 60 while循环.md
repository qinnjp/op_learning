## while循环

***1.什么是while***

while在shell中也是负责循环的语句,和for一样

***2.while和for循环的怎么选***

因为很多功能一样,很多人在学习和工作中的脚本遇到循环到底该使用while还是for呢

1. 知道循环次数使用for,比如一天24次;
2. 如果不知道要循环多少次,那就使用while,比如猜数字游戏,每个人猜对的次数是未知的

***3.while基础语法***

```bash
while 条件测试
do
	循环体
done

i=1
while [ $i -lt 10 ]
do
	echo $i 
	let i++
done
```

```bash
#条件为真一直循环
while true
do
	echo "...."
done
```

**条件比对**

while本质上就是循环
	只不过while支持条件测试语句
		

整数比对

```bash
while [i -lt 10]
```



字符串比对

```bash
read -p "$hostname login: " tt
while [ $tt != "root" ]
do
	read -p "$hostname login: " tt
done
```



正则比对

```bash
read -p "$hostname login: " tt
while [[ ! $tt =~ ^r...$ ]]
do
	read -p "$hostname login: " tt
done
```



文件比对

```bash
while 
do 
	
done</etc/passwd
```



### 示例

需求1: 猜数字游戏 
1)随机输出一个1-100的数字   echo $(($RANDOM%100+1))
2)要求用户输入的必须是数字（数字处加判断）
3)如果比随机数小则提示比随机数小了 大则提示比随机数大了
4)正确则退出 错误则继续死循环
5)最后统计猜了多少次（猜对了多少次，失败多少次)

```bash
[root@manager while]# cat while-14.sh 
#!/bin/bash

SJ=$(($RANDOM%100+1))
i=1

while true
do
	read -p "请输入你要猜的数: " Action

	if [ $Action -eq $SJ ];then
		echo "恭喜你 gdx...."
		break

	elif [ $Action -lt $SJ ];then
		echo "太小了 gdx...."
	else
		echo "太大了 gdx...."
	fi
	let i++
done
	echo "你总共猜了 $i 次, 失败了 $(( $i-1 )) 次"

```

需求2: 在一个2000多行数据文件，然后有10个新的文件，1-5行数据放到第一个文件里，6-10行数据 放到第二个文件里...46-50数据放到第10个文件里。然后51-55数据在放到第一个文件里。如何实现？

```bash
[root@manager while]# cat while-15.sh 
#!/bin/bash

while true
do
	for i in $(seq 10)
	do
		head -5 file.txt >> file_$i.txt
		sed -i '1,5d' file.txt

		if [ ! -s file.txt ];then
			exit
		fi
	done
done
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
#!/bin/bash
a=0
b=0
while [ $b -lt 2000 ]
do
        file=num.txt
        while [ $a -lt 10 ]
        do
                a=$[$a+1]
                b=$[$b+5]
                echo "$a $b"
                line=$(awk "NR==$[$b-4],NR==$b" $file)
                echo "$line">>${a}.txt
        done
        a=0
done
```

