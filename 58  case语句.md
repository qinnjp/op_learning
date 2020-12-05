## case语句

***1.什么是case***

case语句和if类似,也是用来判断,只不过当判断条件较多时,使用case语句会比if更加方便

***2.case使用场景***

在生产环境中我们总会遇到一个问题需要根据不同预案,那么我们要处理这样的问题首先要根据可能出现的情况写出对应的预案,然后根据选择来加载不同的预案

比如服务的启停脚本,我们首先要写好启动,停止重启的预案,然后根据不同的选择来加载不同的预案

***3.case基础语法***

```bash
case 变量 in
条件 1)
	执行代码块1
		;;
条件 1)
	执行代码块2
		;;
条件 1)
	执行代码块3
		;;
条件 1)
	执行代码块4
		;;
    *)
		不匹配后命令序列
esac
```

1. case演示示例:

需求2：使用case实现nginx状态监控脚本。 stub_status 

sh nginx_status.sh

USAGE nginx_status.sh { Active | accepts | handled | requests | Reading  | Writing  |Waiting }

1.nginx开启状态监控

```bash
[root@manager case]# cat /etc/nginx/conf.d/test.conf 
	server {
	listen 80;
	server_name test.oldxu.com;

	location / {
		index index.html;
	}

	location /nginx_status {
		stub_status;
	}
}
```

2.awk取值		curl -H Host:test.oldxu.com http://127.0.0.1:80/nginx_status
		
3.case

```bash
[root@manager case]# cat case-5.sh 
#!/bin/bash

# Author:      Oldux.com QQ: 552408925

# Date：       2019-10-30

# FileName：   case-5.sh

# URL:         https://www.xuliangwei.com

# Description：
HostName=test.oldxu.com
Nginx_status_file=nginx.status
Nginx_Status_Path=nginx_status

curl -sH Host:${HostName} http://127.0.0.1/${Nginx_Status_Path} >${Nginx_status_file}
case $1 in
	active)
		echo $(( $(awk '/Active connections/ {print $NF}' ${Nginx_status_file}) -1 ))
		;;
	accepts)
		echo $(( $(awk 'NR==3 {print $1}' ${Nginx_status_file}) -1 ))
		;;
	handled)
		echo $(( $(awk 'NR==3 {print $2}' ${Nginx_status_file}) -1 ))
		;;
	requests)
		echo $(( $(awk 'NR==3 {print $3}' ${Nginx_status_file}) -1 ))
	;;
*)
	echo "USAGE: $0 { active | accepts | handled | requests | Reading  | Writing  | Waiting }"
	exit 1
esac
```

### Ngixn启停脚本

```bash
#!/bin/bash
# Author:      Oldux.com QQ: 552408925
# Date：       2019-10-30
# FileName：   case-4.sh
# URL:         https://www.xuliangwei.com
# Description： 
source /etc/init.d/functions
nginx_pid=/var/run/nginx.pid


case $1 in
	start)
		if [ -f $nginx_pid ];then
			if [ -s $nginx_pid ];then
				action "Nginx 已经启动" /bin/false
			else
				rm -f $nginx_pid
				systemctl start nginx &>/dev/null
				if [ $? -eq 0 ];then
					action "Nginx启动成功" /bin/true
				else
					action "Nginx启动失败" /bin/false
				fi
			fi
		else
			systemctl start nginx &>/dev/null
			if [ $? -eq 0 ];then
				action "Nginx启动成功" /bin/true
			else
				action "Nginx启动失败" /bin/false
			fi
		fi
		;;
	stop)
		if [ -f $nginx_pid ];then
			systemctl stop nginx && \
			rm -f $nginx_pid
			action "Nginx is stopped" /bin/true
		else
			echo "[error] open() "$nginx_pid" failed (2: No such file or directory)"
		fi
		;;
	status)
		if [ -f $nginx_pid ];then
			echo "nginx (pid  $(cat $nginx_pid)) is running..."
		else
			echo "nginx is stopped"
		fi
		;;
	reload)
		#1.nginx没有启动直接报错
		#2.nginx启动了,先检查配置文件语法
			#如果nginx语法没问题,则reload
			#如何nginx语法有问题
				#提示用户是否进入对应的错误行数修改 [ y | n ]
					# y 进入修改
					# n 退出操作
		if [ -f $nginx_pid ];then
			nginx -t -c /etc/nginx/nginx.conf &>nginx.err
			rc=$?

			if [ $rc -eq 0 ];then
				action "Nginx is Reloaded" /bin/true
			else
				ngx_conf=$(cat nginx.err |awk -F "[ :]" 'NR==1 {print $(NF-1)}')
				ngx_line=$(cat nginx.err |awk -F "[ :]" 'NR==1 {print $NF}')
				read -p "是否进入 ${ngx_conf} 配置文件的 ${ngx_line} 行修改: [ y | n ] " tt
				case $tt in
					y)
						vim $ngx_conf +${ngx_line}
						;;
					n)
						exit 1
				esac
			fi

		else
			action "Nginx 没有启动" /bin/false
		fi
		;;
	*)
		echo "USAGE:  $0 {start|stop|status|restart}"
		exit 1
esac

```



