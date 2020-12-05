## 1.实时同步的概念



- 什么是实时同步

  监控一个目录的变化，当该目录出发事件（创建\删除\修改）

  就执行动作，这个动作可以是rsync同步，也可以是其他

- 为什么要实时同步

  1. 能解决nfs单点故障问题
  2. 能够让本地快速切换至云端（本地机器先保留一个月以免云端发生数据缺失，还可以回退。）

- 实时同步的原理

  它会借助一个通知接口，inotify，inotify监控本地主机的事件（创建\删除\修改）

  则通知执行动作 这个动作可以是rsync同步

  ![1568102511543](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1568102511543.png)

![1568104654061](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1568104654061.png)

- 实时同步工具选择
  1. inotify+rsync实现
  2. sersync实时同步
  3. lsyncd







> 整体：
>
> 我需要监控当前服务器的/data目录，监控创建\删除\修改等事件一旦触发 inotify中的事件
>
> 则执行rsync命令，推送给远程的172.16.1.41的data的模块

~~~php
[root@nfs01 sersync]# vim confxml.xml                            
  5     <fileSystem xfs="true"/>  <!-- 文件系统 -->                           
  6     <filter start="false">  <!-- 排除不想同步的文件-->
  7         <exclude expression="(.*)\.svn"></exclude>
  8         <exclude expression="(.*)\.gz"></exclude>
  9         <exclude expression="^info/*"></exclude>
 10         <exclude expression="^static/*"></exclude>
 11     </filter>
 
 12     <inotify> <!-- 监控的事件类型 -->
 13         <delete start="true"/>     
 14         <createFolder start="true"/>    #创建目录
 15         <createFile start="true"/>      #创建文件
 16         <closeWrite start="true"/>      
 17         <moveFrom start="true"/>
 18         <moveTo start="true"/>
 19         <attrib start="false"/>
 20         <modify start="false"/>
 21     </inotify>
 
 23     <sersync>
 24         <localpath watch="/data"> <!-- 监控的目录 -->
 25             <remote ip="172.16.1.41" name="data"/>  <!-- backup的IP以及模块 -->
 28         </localpath>
 
 29         <rsync> <!-- rsync的选项 -->
 30             <commonParams params="-avz"/>
 31             <auth start="true" users="rsync_backup" passwordfile="/etc/rsync.pass"/>
 32             <userDefinedPort start="false" port="874"/><!-- port=874 -->
 33             <timeout start="true" time="100"/><!-- timeout=100 -->
 34             <ssh start="false"/>
 35         </rsync>

            <!-- 每60分钟执行一次同步-->
  36         <failLog path="/tmp/rsync_fail_log.sh" timeToExecute="60"/><!--def
    ault every 60mins execute once-->
~~~



![1568108354360](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\1568108354360.png)