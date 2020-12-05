## 1.Awk基础介绍

***1.什么是awk***
awk不仅仅是一个文本处理工具,通常用于处理数据并生成结果报告。当然awk也是一门编程语言，是Iinux上功能最强大的数据处理工具之一。
***2.awk语法格式***
第一种形式:`awk 'BEGIN{} pattern {commands} END {}' file_name`
第二种形式:`standard output| awk BEGIN{} pattern { commands} END {}`
第三种形式: `awk [options] -f awk-script-file filenames`

| 语法格式   | 含义                     |
| ---------- | ------------------------ |
| BEGIN {}   | 正式处理数据之前执行     |
| pattern    | 匹配模式                 |
| {commands} | 处理命令, 可能多行       |
| END{}      | 处理完所有匹配数据后执行 |

***3.语法格式示例***

![1573093711763](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1573093711763.png)

含义解释:
BEGIN发现在读文件之前,所有会在处理之前就执行了1/2。{}表示处理文件的过程,由于文件内有三行,所以会执行三次print。{END}表示文件处理完毕后的动作。

#### 1.awk工作原理

`#awk -F: '{print $1,$3}' /etc/passwd`

1.awk将文件中的每一行作为输入, 并将每一行赋给内部变量$0, 以换行符结束
2.awk开始进行字段分解，每个字段存储在已编号的变量中，从$1开始[默认空格分割]
3.awk默认字段分隔符是由内部FS变量来确定, 可以使用-F修订
4.awk行处理时使用了print函数打印分割后的字段

5.awk在打印后的字段加上空格，因为$1,$3 之间有一个逗号。逗号被映射至OFS内部变量中，称为输出字段分隔符， OFS默认为空格.
6.awk输出之后，将从文件中获取另一行，并将其存储在$0中，覆盖原来的内容，然后将新的字符串分隔成字段并进行处理。该过程将持续到所有行处理完毕.

![1573094951522](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1573094951522.png)

#### 2.Awk内部变量

| 内置变量 | 含义                                        |
| -------- | ------------------------------------------- |
| $0       | 整行内容                                    |
| $1-$n    | 当前行的第1-n个字段                         |
| NF       | 当前行的字段个数,也就是多少列               |
| NR       | 当前的行号,从1开始计数                      |
| FS       | 输入字段分隔符。不指定默认以空格或tab键分割 |
| RS       | 输入行分隔符。默认回车换行                  |
| OFS      | 输出字段分隔符。默认为空格                  |
| ORS      | 输出行分隔符。默认为回车换行                |

要想了解awk的一些内部变量需要先准备如下数据文件。

```bash
[root@o1dxu ~]# cat awk_ file. txt
ll 1990 50 51 61
kk 1991 60 52 62
hh 1992 70 53 63
jj 1993805464
mm 1994 90 55 65
```

1.awk内置变量, $0保存当前记录的内容

```bash
[root@oldxu ~]# awk '{print $0}' /etc/passwd
```

2.awk内置变量, FS指定字段分割符，默认以空白行作为分隔符

```bash
#1.给出文件第一列
[root@ssh02 awk_2]# awk '{print $1}' awk-file.txt 
ll
kk
hh
jj
mm

#2.指定多个分隔符,获取第一列内容
[root@ssh02 awk_2]# cat awk-file.txt 
ll:1990 50 51 61
kk:1991 60 52 62
hh:1992 70 53 63
jj:1993805464
mm 1994 90 55 65

[root@ssh02 awk_2]# awk 'BEGIN {FS="[: ]"} {print $1}' awk-file.txt 
ll
kk
hh
jj
mm

`[: ]+连续的冒号当做一个分隔符,连续的控格也当做一个分隔符,连续的冒号和空格也当做一个分隔符`
```

3.awk内置变量NF,保存每行的最后一列

```bash
[root@ssh02 awk_2]# awk '{print NF,$NF}' awk-file.txt 
4 61
4 62
4 63
4 64
5 65
```

4.AWK内置变量NR,表示记录行号.

```bash
#NR会记录每一行的行号
[root@ssh02 awk_2]# awk '{print NR,$0}' awk-file.txt 
1 ll:1990 50 51 61
2 kk:1991 60 52 62
3 hh:1992 70 53 63
4 jj:1993 80 54 64
5 mm 1994 90 55 65

[root@ssh02 awk_2]# awk 'NR>1 && NR<4 {print NR,$0}' awk-file.txt 
2 kk:1991 60 52 62
3 hh:1992 70 53 63
```

5.awk内置变量FS,默认以空格和制表符为字段分隔符

```bash
[root@ssh02 awk_2]# awk 'BEGIN{FS=":"}{print $1}' awk-file.txt 
ll
kk
hh
jj
```

6.awk内置变量RS,行分隔符

```bash
[root@ssh02 awk_2]# cat awk-file.txt 
ll--1990--50--51--61

[root@ssh02 awk_2]# awk 'BEGIN {RS="--"}{print $0}' awk-file.txt 
ll
1990
50
51
61

```

7.awk内置变量,OFS指定输出字段分隔符,初始情况下OFS变量是空格

```bash
[root@ssh02 awk_2]# cat awk-file.txt 
l|l--19|90--5|0--5|1--6|1

[root@ssh02 awk_2]# awk 'BEGIN {RS="--";FS="|";OFS=":"}{print $1,$2}' awk-file.txt 
l:l
19:90
5:0
5:1
6:1
```

8.awk内置变量,ORS指定输出行分隔符,默认行分隔符为\n

```bash
[root@ssh02 awk_2]# awk 'BEGIN {RS="--";FS="|";OFS=":";ORS="---"}{print $1,$2}' awk-file.txt 
l:l---19:90---5:0---5:1---6:1
```

#### 3.AWK格式输出

awk可以通过printf函数生成非常漂亮的数据报表

| 格式符 | 含义           |
| ------ | -------------- |
| %s     | 打印字符串     |
| %d     | 打印十进制数   |
| %f     | 打印一个浮点数 |
| %x     | 打印十六进制数 |
| %o     | 打印八进制数   |
| 修饰符 | 含义           |
| -      | 左对齐         |
| +      | 右对齐         |

1.printf默认没有换行符

```bash
[root@ssh02 ~]# awk 'BEGIN{FS=":"} {printf $1}' /etc/passwd
rootbindaemonadmlpsyncshutdownhaltmailoperatorgamesftpnobodysystemd-networkdbuspolkitdtssabrtsshdpostfixchronyrpcrpcusernfsnobodyapachezabbixmysqlqjpnginx
```

2.加入换行,格式化输出

```bash
[root@ssh02 ~]# awk 'BEGIN{FS=":"} {printf  "%s\n",$1}' /etc/passwd
root
bin
daemon
adm
lp
sync
```

3.使用占位符美化输出,"-"表示左对齐

```bash
[root@ssh02 ~]# awk 'BEGIN{FS=":"} {printf  "%-20s %-20s\n ",$1,$7}' /etc/passwd
root                 /bin/bash           
 bin                  /sbin/nologin       
 daemon               /sbin/nologin       
 adm                  /sbin/nologin       
 lp                   /sbin/nologin       
 sync                 /bin/sync           
 shutdown             /sbin/shutdown      
 halt                 /sbin/halt          
 mail                 /sbin/nologin       
 operator             /sbin/nologin   
```

示例(追加列名称)

```bash
[root@manager awk]# awk '
BEGIN { 
	printf "%-20s%-20s%-20s%-20s\n",
	"Name","shuxue","yuwen","yinx"
} 
{
	printf "%-20s%-20s%-20s%-20s\n",  $1,$2,$3,$4
}' file3.txt

Name                shuxue              yuwen               yinx                
Oldxu               20                  30                  40                  
oldqiang            10                  5                   2                   
oldguo              1                   1                   1                   
oldgao              1                   2                   3                   
oldboy              10                  2                   0  
```



#### 4.awk模式匹配

awk第一种模式匹配:RegExp

awk第二种模式匹配:关系运算符匹配

***RegExp示例***

1.匹配/etc/passwd文件中以root开头的行

```bash
[root@ssh02 ~]#awk 'BEGIN {FS=":"} /^root/{print $0}' /etc/passwd
[root@ssh02 ~]#awk '/^root/' /etc/passwd
```

***布尔运算符匹配示例***

| 符号 | 含义             |
| ---- | ---------------- |
| <    | 小于             |
| >    | 大于             |
| <=   | 小于等于         |
| > =  | 大于等于         |
| ==   | 等于             |
| !=   | 不等于           |
| ~    | 匹配正则表达式   |
| !~   | 不匹配正则表达式 |

1.以:为分隔符匹配passwd文件中包含ftp或mail的所有行信息

```bash
[root@ssh02 ~]# awk 'BEGIN {FS=":"}$1=="ftp" || $1=="mail"{print $0}' /etc/passwd
```

2.匹配passwd文件中第三列小于50且第四列大于50的所有行信息

```bash
[root@ssh02 ~]# awk 'BEGIN {FS=":"}$3<50 && $4>50 {print $0}' /etc/passwd
```

3.匹配passwd文件中最后一列匹配bin/bash的行

```bash
完全匹配/bin/bash
[root@ssh02 ~]# awk 'BEGIN {FS=":"}$7=="/bin/bash"{print $0}' /etc/passwd
包含/bin/bash
[root@ssh02 ~]# awk 'BEGIN {FS=":"}$7~"/bin/bash"{print $0}' /etc/passwd
```

4.匹配passwd文件中最后一列不匹配bin/bash的行

```bash
[root@ssh02 ~]# awk 'BEGIN {FS=":"}$7!="/bin/bash"{print $0}' /etc/passwd
```

5.匹配/etc/passwd文件中第三个字段包含3个数字以上的所有信息

```bash
[root@ssh02 ~]# awk 'BEGIN {FS=":"}$3 ~ /[0,9]{3,}/{print $0}' /etc/passwd
```

| 符号 | 含义 |
| ---- | ---- |
| \|\| | 或   |
| &&   | 与   |
| !    | 非   |

1、以:为分隔符，匹配passwd文件中包含ftp或mail的所有行信息。

```bash
[root@oldxu ~]# awk 'BEGIN{FS=":"}$1=="ftp" || $1=="mail" {print $0}' passwd
```

2、以:为分隔符，匹配passwd文件中第3个字段小于50并且第4个字段大于50的所有行信息。

```bash
[root@oldxu ~]# awk 'BEGIN{FS=":"}$3<50 && $4>50{print $0}' passwd
```

3.匹配没有/sbin/nologin 的行。

```bash
[root@oldxu ~]# awk 'BEGIN{FS=":"} $0 !~ /\/sbin\/nologin/{print $0}' passwd
[root@manager awk]# awk 'BEGIN{FS=":"} $7 != "/sbin/nologin"' passwd 
```

运算符匹配示例(支持小数)

| 运算符 | 含义 |
| ------ | ---- |
| +      | 加   |
| -      | 减   |
| *      | 乘   |
| /      | 除   |
| %      | 余   |

```bash
2.计算下列每个同学的平均分数，并且只打印平均分数大于90的同学姓名和分数信息
[root@oldxu ~]# cat student.txt
oldxu       80    90    96    98
oldqiang    93    98    92    91
oldguo      78    76    87    92
oldli       86    89    68    92
oldgao      85    95    75    90

[root@ssh02 awk]# cat tongji.awk 
BEGIN{
	printf "%-10s%-10s%-10s%-10s%-10s%-10s%-10s\n",
	"Name","yuwen","shuxue","yingyu","it","Total","AVG"
}
{
	Total=$2+$3+$4+$5
	AVG=($2+$3+$4+$5)/4
	if (AVG>90) {
	printf "%-10s%-10d%-10d%-10d%-10d%-10d%-10.2f\n",
	$1,$2,$3,$4,$5,Total,AVG
	}
}

```





#4.指定多个分隔符
开叶.1日心夕1 1M217V
[root@o1dxu ~]# cat awk_ file. txt
1l: :1990 50 51 61
kk:1991 60 52 62
hh 1992 70 53 63
:100900CACn
mm 1994 90 55 65
#[: ]+连续的多个冒号当-一个分隔符，连续的多个空格当一个分隔符， I连续空格和冒号也当做一个字符来处理
[root@oldxu ~]# awk -F '[: ]+' '{print $2}' awk_ file. txt
1990
1991
1992
1993
**1994**





5.Awk模式匹配
awk第一种模式匹配: RegExp
awk第二种模式匹配:关系运算四配的
RegExp示例
1.匹配etc/passwd文件行中含有root字符串的所有行。
[root@oldxu ~]# awk ' BEGIN{FS=": "}/root/{print $e}' passwd
2.匹配etc/passwd文件行中以root开头的行。

![1573126015384](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1573126015384.png)