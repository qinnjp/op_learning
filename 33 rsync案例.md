## 案例

已知3台服务器主机名分别为web01、backup 、nfs主机信息见下表:

角色 外网IP(NAT) 内网IP(LAN) 主机名
WEB	eth0:10.0.0.7	eth1:172.16.1.7	web01
NFS	eth0:10.0.0.31	eth1:172.16.1.31	nfs
Rsync	eth0:10.0.0.41	eth1:172.16.1.41	backup


客户端: web nfs
服务端: backup

 

**客户端需求**
1.客户端提前准备存放的备份的目录，目录规则如下:/backup/nfs_172.16.1.31_2018-09-02 
2.客户端在本地打包备份(系统配置文件、应用配置等)拷贝至/backup/nfs_172.16.1.31_2018-09-02
3.客户端最后将备份的数据进行推送至备份服务器
4.客户端服务器本地保留最近7天的数据, 避免浪费磁盘空间
4.客户端每天凌晨1点定时执行该脚本

**服务端需求**
1.服务端部署rsync，用于接收客户端推送过来的备份数据
2.服务端需要每天校验客户端推送过来的数据是否完整
3.服务端需要每天校验的结果通知给管理员
4.服务端仅保留6个月的备份数据,其余的全部删除

注意：所有服务器的备份目录必须都为/backup

快递: 散货-------->打包--->标记------->装车-------------->运输------------->仓库
         验货-------->评价

系统: 备份的文件-->打包--->校验值----->备份至本地目录---->网络-------->推送至备份服务器
         管理员校验-->通知--->邮件

### ***脚本过程：***



***客户端***

~~~bash
[root@nfs ~]# mkdir /scripts
[root@nfs ~]# cat /scripts/clinet_push_data_server.sh 
#!/usr/bin/bash
# variables == 变量  ---> 一个固定的字符串表示一个不固定的值
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
Src=/backup
Host=$(hostname)
Addr=$(ifconfig eth1 | awk 'NR==2 {print $2}')
Date=$(date +%F)
Dest=${Host}_${Addr}_${Date}

#1.准备对应的备份目录
[ -d $Src/$Dest ] ||  mkdir -p $Src/$Dest

#2.将文件拷贝至备份目录
cd / && \
[ -f $Src/$Dest/sys.tar.gz ] ||  tar czf $Src/$Dest/sys.tar.gz etc/fstab etc/hosts etc/passwd && \
[ -f $Src/$Dest/other.tar.gz ] || tar czf $Src/$Dest/other.tar.gz var/spool/cron/ scripts/ && \

#3.添加标记
[ -f $Src/$Dest/flag_${Date} ] ||  md5sum $Src/$Dest/*.tar.gz > $Src/$Dest/flag_${Date}

#4.推送数据至远程仓库
export RSYNC_PASSWORD=123456
rsync -avz $Src/ rsync_backup@172.16.1.41::backup

#5.保留本地最近7天的数据
find $Src/ -type d -mtime +7 | xargs rm -rf
	

#批量的模拟数据
[root@nfs ~]# for i in {1..30};do date -s "201909$i"; sh /scripts/clinet_push_data_server.sh ; done
~~~

***服务端***

~~~bash
#1.服务端配置邮件功能
[root@backup /]# yum install mailx -y
[root@backup /]# vim /etc/mail.rc		#跳转至最后一行,然后进入编辑模式
set from=发件人@qq.com
set smtp=smtps://smtp.qq.com:465
set smtp-auth-user=发件人@qq.com
set smtp-auth-password=xxxx
set smtp-auth=login
set ssl-verify=ignore
set nss-config-dir=/etc/pki/nssdb/


#2.测试一下是否能发送成功
mail -s "测试一下" 收件人@qq.com < /etc/hosts

#3.编写脚本
[root@backup ~]# cat /scripts/check_data_notify.sh 
#!/usr/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
Src=/backup
Date=$(date +%F)

#1.校验每天客户端推送过来的flag数据
md5sum -c ${Src}/*_${Date}/flag_${Date} >${Src}/result_${Date}

#2.邮件通知管理员
mail -s "Rsync Backup ${Date}" 收件人@qq.com < ${Src}/result_${Date}

#3.保留最近180天的数据
find $Src/ -type d -mtime +180 | xargs rm -rf

~~~

***定时任务***

客户端

~~~bash
[root@nfs ~]# crontab -l
#定时备份数据
*/1 * * * * sh /scripts/clinet_push_data_server.sh &>/dev/null
~~~

服务端

~~~bash
[root@backup ~]# crontab -l
#定时校验备份的结果
*/1 * * * * sh /scripts/check_data_notify.sh &>/dev/null
~~~

如何在增加一台客户端备份

~~~bash
[root@web01 ~]# rsync -avz root@172.16.1.31:/scripts /
~~~











​        