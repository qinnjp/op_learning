## sed文本处理工具

#### 1.sed基本概述

***1.sed基本介绍***

sed(Stream Editor),流编辑器。能够对标准输出或文件进行逐行处理。

简单来说, sed可以实现对文件的增删查改。
***2.sed.工作模式***

sed读取文件一行,存放在缓存区,然后处理,最后输出。

***3.sed基础语法***

第一种形式: `stdout | sed [option] "pattern command"`

第二种形式: `sed [option] "pattern command" file`

#### 2.sed常用选项

| 选项 | 含义                               |
| ---- | ---------------------------------- |
| -n   | 只打印匹配的行(取消文件的默认输出) |
| -e   | 允许多项编辑                       |
| -f   | 编辑动作保存在文件,指定文件才执行  |
| -r   | 支持扩展正则表达式                 |
| -i   | 直接变更文件内容                   |

0. sed相关示例文件

```bash
[ root@web01 opt]# cat file. txt
I love shell
I love SHELL
This is test file
```

1.sed-n、-e选项示例

```bash
#取消默认输出
[root@web01 opt]# sed -n '/shell/p' file. txt
I love shell
#编辑多项
[root@web01 opt]# sed -n -e '/shgll/p' -e '/SHELL/p' file. txt
I love shell
I love SHELL
```

2. sed -f选项示例

```bash
#将pattern写入文件中
[root@web01 opt]# cat edit.sed
/she1l/p
[root@web01 opt]# sed -n -f edit.sed file. txt
```

3.sed -r选项示例

```bash
[root@web01 opt]# sed -n '/sheLL/SHELL/p’ file. txt
#扩展正则表达式
[root@web01 opt]# sed -rn '/sheLl/SHELL/p' file. txt
I love shell
I love SHELL
```

4.sed -i选项

```bash
[ root@manager sed]# sed -in ' /shell /d' file. txt
```



#### 3.sed pattern

命令格式:`sed [option] ' /pattern/ command' file`

| 匹配模式                     | 含义                                           |
| ---------------------------- | ---------------------------------------------- |
| 10command                    | 匹配第10行                                     |
| 10,20command                 | 匹配从第10行开始，到第20行结束                 |
| 10, +5command                | 匹配从第10行开始,到第16行结束                  |
| /pattern1/,/pattern2/command | 匹配到pattern1的行开始,到匹配到pattrn2的行结束 |
| 10,/pattern1/command         | 匹配从第10行开始,到匹配到pattern1的行结束      |

1.示例:指定行号。

```bash
#打印passwd文件的第10行
[root@web01 opt]# sed -n '10p' passwd
```

2.示例指定起始行号和结束行号

````bash
#打印passwd.文件的1日到2日行
[root@web01 opt]# sed -n '10,20p' passwd
````

3.指定起始行号,然后后面N行

```bash
#打Epasswd文件中从第1行开始，往后面加5行的内容
[root@web01 opt]# sed -n '1,+5p' passwd
```

4.正则表达式匹配

```bash
#打印passwd文件中以root开头的行
[root@web01 opt]# sed -n '/^root/p' passwd

#打印passwd文件第一个匹配到以bin开头的行， 到第二个匹配到以ftp的行
[root@web01 opt]# sed -n '/^bin/,/^ftp/p' passwd
```

6.从指定的行号开始匹配,直到匹配到pattern 1的行

```bash
#打印passwd文件中从第2行开始匹配，直到以^halt开头的行结束
[root@web01 opt]# sed -n '2, /^halt/p' passwd
```



#### 4.sed追加

| 编辑命令 | 含义                  |
| -------- | --------------------- |
| a        | 后追加内容            |
| i        | 行前追加内容          |
| r        | 读入外部文件,行后追加 |
| w        | 将匹配行写入外部文件  |

1.匹配bin/bash的行,在其行后面添加一行内容。

```bash
[root@web01 opt]# sed -i '/^bin/a OK' passwd
```

2.以/bin开头的行到以sshd开头的行,前面添加一行。

```bash
[root@web01 opt]# sed '/^bin/,/^sshd/i AAA-AAA-OK' passwd
```

3.指定给文件的30行添加一行内容。

```bash
[root@She1l ~]# sed -i '30i listen 80;' passwd
```

4.将list. txt文件中的内容,追加到匹配模式的行后面

```bash
[root@She1l ~]# sed -i '/root/r List. txt' passwd
```

5.匹配bin/bash所有的行,将其保存至/tmp//ogin.txt文件中

```bash
[root@She1l ~]# sed '/\/bin\/bash/w /tmp/Login. txt ' passwd
```



#### 5.sed删除命令

| 编辑命令             | 含义                                                 |
| -------------------- | ---------------------------------------------------- |
| 1d                   | 删除第1行的内容                                      |
| 1,5d                 | 删除1行到5行的内容                                   |
| 2, +5d               | 删除2行以及往下的5行的内容                           |
| /pattern1/d          | 删除每行中匹配到pattern1的行内容                     |
| /pattern1/pattern2/d | 删除匹配到pattern1的行直到匹配到pattern2的所有行内容 |
| /pattern1/, 10d      | 删除匹配到pattern1的行到10行的所有行内容             |
| 10,/pattern1/d       | 删除第10行直到匹配到pattern1的所有内容               |

1.删除passwd文件中第1行内容

```bash
[root@web01 opt]# sed '1d' passwd
```

2.删除passwd文件中第1行到第5行的内容

```bash
[root@web01 opt]# sed '1,5d' passwd
```

3.删除passwd文件中第2行以及往下的5行内容

```bash
[root@web01 opt]# sed '2, +5d' passwd
```

4.匹配sbin/nologin结尾的行,然后进行删除。

```bash
[root@web01 opt]# sed '/\/sbin \/nologin$/d' passwd
```

5.匹配以sshd开头的行,到rpc开头的行

```bash
[root@webe1 opt]# sed '/^sshd/,/^rpc/d" passwd
```

6.删除vsttpd配置文件以#号开头的行,以及空行

```bash
#删除配置文件中#号开头的注释行，如果碰到tab或空格是无法删除
[root@Shell ~]# sed '/^#/d' file
#删除配置文件中含有tab键的注释行
[root@Shell ~]# sed -r '/^[ \t]*#/d' file
#删除无内容空行
[root@Shell ~]# sed -r '/^[ \t]*$/d' file
#并删除vsftpd配置文件的注释行以及空行
[root@She1l ~]# sed -r '/^[ \t]*#/d; /^[ \t]*$/d' /etc/vsftpd/vsftpd. conf
[root@She1l ~]# sed -r '/^[ \t]*#|^[ \t]*$/d' /etc/vsftpd/vsftpd. conf
[root@She1l ~]# sed -r '/^[ \t]*($|#)/d' /etc/vsftpd/vsftpd. conf
```



#### 6.sed修改命令

| 编辑命令                       | 含义                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| 1 s/old/new/                   | 替换第1行内容old为new                                        |
| 1, 10s/old/new/                | 替换1行到10行的内容old为new                                  |
| 1,+5s/old/new/                 | 替换1行到6行的内容old为new                                   |
| /pattern1/s/old/new/           | 替换匹配pattern1的内容old为new                               |
| /pattern1/,/pattern2/s/old/new | /替换匹配到pattern1的行直到匹配到pattern2的所有行内容old为new |
| 10,/pattern 1/s/old/new/       | 替换第10行直到匹配到pattern1的所有行内容old为enw             |

1.修改passwd文件第1行中第一个root为ROOT

```bash
sed -i  '1s/root/ROOT/' passwd
```

2.修改passwd文件中第5行到第10行中所有的/sbin/nologin为/bin/bash

```bash
sed -i '5,10s/\/sbin\/nologin/\/bin\/bash/' passwd
sed -i '5,10s#/sbin/nologin#/bin/bash#' passwd
```

3.修改passwd文件中匹配到/sbin/nologin的行，将匹配到行中的login为该大写的LOGIN

```bash
sed -i '/\/sbin\/nologin/s#login#LOGIN#g' passwd
sed -i '/\/sbin\/nologin/s/login/LOGIN#g' passwd
```

4.修改passwd文件中从匹配到以root开头的行，到匹配到以bin开头行,修改bin为BIN

```bash
sed -i '/^root/,/^bin/s/bin/BIN/g' passwd
```

5.修改SELINUX=enforcing修改为SELINUX=disabled。（可以使用c替换方式）

```bash
sed -i '/^SELINUX=/c SELINUX=disabled' selinux
```

6.将nginx.conf配置文件添加注释。  ^   $

```bash
sed -i 's/^/# /' nginx.conf
```

7.使用sed提取eth0网卡的IP地址

```bash
ifconfig eth0 | sed -rn '2s/^.*et //p' | sed -rn 's/ ne.*//p' 
ifconfig eth0 |sed -nr '2s/(^.*et) (.*)(net.*)/\2/p'
```

7.sed脚本练习
	需求描述：处理一个类似MySQL配置文件的my.cnf文件。
	1.输文件中有几个段，一对 [ ] 为一段。
	2.针对每个段配置文件参数，统计总个数。
	[root@web01 opt]# sh example.sh
	1: client 2
	2: server 12
	3: mysqld 12
	4: mysqld_safe 7
	5: embedded 8
	6: mysqld-5.5 10

```bash
[root@manager~/sed]# cat sed_b2c.sh 
#!/bin/bash
Sed_inventory () {
	B2c=$(sed -n '/\[*\]/p' b2c-inventory |sed 's/^\[*//'|sed 's/\]//')
	echo $B2c
}
sed_b2c () {
	sed -n '/^\['$item'\]/,/^\[.*\]/p' b2c-inventory |sed -r '/\[|^$/d'|wc -l
}
index=0
for item in $(Sed_inventory)
do
	let index++
	echo $index:$item $(sed_b2c)
done
```





