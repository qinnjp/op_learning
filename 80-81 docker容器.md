

## docker容器

#### 1 什么是容器

容器就是在隔离的环境运行的一个进程，如果进程停止，容器就会销毁。隔离的环境拥有自己的系统文件，ip地址，主机名等。

#### 2 容器和虚拟化的区别

- linux容器技术，容器虚拟化和kvm虚拟化的区别
  - kvm虚拟化：需要硬件的支持，需要模拟硬件，可以运行不同的操作系统，启动时间分钟级(开机启动流程)
  - 容器：共用宿主机内核，第一个进程直接启动服务(nginx，httpd，mysql)
- 容器：共用宿主机内核，轻量级，损耗少，启动快，性能高，只能运行在linux系统上
- 虚拟机：需要硬件的支持，需要模拟硬件，需要走开机启动流程，可以运行不同的操作系统

![1575372874601](F:\typora\1575372874601.png)

#### 3 容器技术的发展过程

***1.chroot技术，新建一个子系统（拥有自己完整的系统文件）***

- 参考资料：

https://www.ibm.com/developerworks/cn/linux/l-cn-chroot/

```bash
chroot 子系统路径   #进入子系统
```



- 作业1：使用chroot监狱限制SSH用户访问指定目录和使用指定命令(cp,ls)
  https://linux.cn/article-8313-1.html

***2.linux容器(lxc) linux container(namespaces 命名空间 隔离环境 及cgroups 资源限制)***

​	cgroups 限制一个进程能够使用的资源。cpu，内存，硬盘io

​	kvm虚拟机：资源限制（1c 1G 20G）

```bash
##需要使用epel源 
#安装epel源 
yum install epel-release -y
```

编译epel源配置文件

```bash
vi /etc/yum.repos.d/epel.repo 
[epel] 
name=Extra Packages for Enterprise Linux 7 - $basearch 
baseurl=https://mirrors.tuna.tsinghua.edu.cn/epel/7/$basearch 
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch 
failovermethod=priority 
enabled=1 
gpgcheck=1 
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo] 
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug 
baseurl=https://mirrors.tuna.tsinghua.edu.cn/epel/7/$basearch/debug 
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-7&arch=$basearch 
failovermethod=priority 
enabled=0 
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7 
gpgcheck=1

[epel-source] 
name=Extra Packages for Enterprise Linux 7 - $basearch - Source 
baseurl=https://mirrors.tuna.tsinghua.edu.cn/epel/7/SRPMS 
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=$basearch 
failovermethod=priority 
enabled=0 
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7 
gpgcheck=1 
```

安装lxc

```bash
yum install lxc-* -y 
yum install libcgroup* -y 
yum install bridge-utils.x86_64 -y 
```

桥接网卡

```bash
[root@controller ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0 
echo 'TYPE=Ethernet 
BOOTPROTO=none 
NAME=eth0 
DEVICE=eth0 
ONBOOT=yes 
BRIDGE=virbr0' >/etc/sysconfig/network-scripts/ifcfg-eth0

[root@controller ~]# cat /etc/sysconfig/network-scripts/ifcfg-virbr0 
echo 'TYPE=Bridge 
BOOTPROTO=static 
NAME=virbr0 
DEVICE=virbr0 
ONBOOT=yes 
IPADDR=10.0.0.11 
NETMASK=255.255.255.0 
GATEWAY=10.0.0.254 
DNS1=180.76.76.76' >/etc/sysconfig/network-scripts/ifcfg-virbr0 
```

启动

```bash
systemctl start cgconfig.service 
```

启动lxc

```bash
systemctl start lxc.service 
```

创建lxc容器

```bash
方法1: 
lxc-create -t download -n centos6 -- --server mirrors.tuna.tsinghua.edu.cn/lxc-images -d centos -r 6 -a amd64 
方法2： 
lxc-create -t centos -n test 
```

为lxc容器设置root密码：

```bash
[root@controller ~]# chroot /var/lib/lxc/test/rootfs passwd 
Changing password for user root. 
New password: 
BAD PASSWORD: it is too simplistic/systematic 
BAD PASSWORD: is too simple 
Retype new password: 
passwd: all authentication tokens updated successfully. 
```

为容器指定ip和网关

```bash
vi /var/lib/lxc/centos7/config 
lxc.network.name = eth0 
lxc.network.ipv4 = 10.0.0.111/24 
lxc.network.ipv4.gateway = 10.0.0.254 
```

启动容器

```bash
lxc-start -n centos7 
```

**lxc 和 docker的区别**

`lxc和docker都要使用namespace资源隔离和cgroup资源限制的技术，但是lxc启动时会启动系统的所有进程，而docker只启动业务进程`

***3.docker容器***

​	Docker是通过进程虚拟化技术（namespaces及cgroups cpu、内存、磁盘io等）来提供容器的资源隔离与安全保障等。由于Docker通过操作系统层的虚拟化实现隔离，所以Docker容器在运行时，不需要类似虚拟机（VM）额外的操作系统开销，提高资源利用率。

`docker 是一个容器引擎 早期的底层是lxc，后来自己开发了libcontainer 容器技术`

#### 4 docker的安装

10.0.0.11

```bash
rm -fr /etc/yum.repos.d/local.repo
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
sed -i 's#download.docker.com#mirrors.tuna.tsinghua.edu.cn/docker-ce#g' /etc/yum.repos.d/docker-ce.repo
yum install docker-ce -y
```

#### 5 docker的主要组成部分

docker是传统的CS架构分为docker client和docker server,向mysql一样

```bash
[root@controller ~]# docker version 
Client: 
Version:    17.12.0-ce 
API version:    1.35 
Go version: go1.9.2 
Git commit: c97c6d6 
Built:  Wed Dec 27 20:10:14 2017 
OS/Arch:    linux/amd64

Server: 
Engine: 
Version:    17.12.0-ce 
API version:    1.35 (minimum version 1.12) 
Go version: go1.9.2 
Git commit: c97c6d6 
Built:  Wed Dec 27 20:12:46 2017 
OS/Arch:    linux/amd64 
Experimental:   false 
```

docker主要组件有：镜像、容器、仓库、网络、存储、监控。

启动容器必须需要一个镜像，docker官方仓库是最大的

容器--镜像--仓库

#### 6 启动第一个容器

docker的主要目标是"Build,Ship and Run any App,Angwhere",构建，运输，处处运行

```bash
#创建并运行一个镜像
docker run -d -p 80:80 nginx
run				#创建并运行一个容器）
-d 				#放在后台
-p 				#端口映射
nginx docker	#镜像的名字
```

#### 7 docker的镜像管理

- 搜索一个镜像

```bash
docker search 镜像名称
```

- 选镜像的建议：
  - 优先考虑官方
  - stars数量多

官方镜像仓库地址：hub.docker.com

```bash
#获取镜像：
docker pull（push）
```

镜像加速器：阿里云加速器，daocloud加速器，中科大加速器，Docker 中国官方镜像加速：https://registry.docker-cn.com

- 配置中国官方镜像加速器

```bash
vi /etc/docker/daemon.json 
{ 
"registry-mirrors": ["https://registry.docker-cn.com"]
} 
```

- 镜像相关命令

```bash
#查看镜像列表
docker images
docker image ls 

#删除镜像
docker image rm 
docker rmi

#导出镜像
docker image save centos -o docker-centos7.4.tar.gz 

#导入镜像
docker image load -i docker-centos7.4.tar.gz 
```

#### 8 docker的容器管理

```bash
#创建并运行一个容器
docker run -d -p 80:80 nginx:latest 
run			#创建并运行一个容器
-d 			#放在后台
-p 			#端口映射
-v 			#源地址(宿主机):目标地址(容器)

#运行并进入一个容器
docker run -it --name centos6 centos:6.9 /bin/bash
-it 			#分配交互式的终端interactive tty
--name 			#指定容器的名字
/bin/bash		#覆盖容器的初始命令
ctrl+p ctrl+q   #将容器放到后台

#运行容器
docker run image_name  
docker container run
docker run ==== docker create + docker start

启动容器
docker start
停止容器
docker stop CONTAINER_ID
杀死容器
docker kill container_name
查看容器列表
docker ps -a,-l,-q

#进入正在运行的容器
docker exec -it 容器id或容器名字 /bin/bash（/bin/sh）
docker attach（使用同一个终端） 偷偷离开的快捷键ctrl+p,ctrl+q

#删除容器
docker rm
#批量删除容器
docker rm -f `docker ps -a -q`

docker logs		#查看容器输入（排错）

#开机自启
--restart=always
```

总结：docker容器内的第一个进程（初始命令）必须一直处于前台运行的状态（必须夯住），否则这个容器，就会处于退出状态！

业务在容器中运行：初始命令,夯住，启动服务

#### 9 docker容器的网络访问（端口映射）

```bash
#指定映射(docker 会自动添加一条iptables规则来实现端口映射) 
-p hostPort:containerPort 
-p ip:hostPort:containerPort 多个容器都想使用8080端口 
-p ip::containerPort(随机端口) 
-p hostPort:containerPort/udp 
-p 10.0.0.100::53/udp 使用宿主机的10.0.0.100这个ip地址的随机端口的udp协议映射容器的udp53端口 
-p 81:80 –p 443:443 可以指定多个-p

随机映射
docker run -P （随机端口）

通过iptables来实现的端口映射
```

#### 10 docker的数据卷管理

```bash
数据卷(文件或目录) 
-v 卷名:/data (第一次卷是空,会容器的数据复制到卷中,如果卷里面有数据,把卷数据的挂载到容器中) 
-v src（宿主机的目录）:dst（容器的目录） 
数据卷容器 
--volumes-from（跟某一个已经存在的容器挂载相同的卷） 
```

基于nginx启动一个容器，监听80和81，访问80，出现nginx默认欢迎首页，访问81，出现小鸟。

```bash
docker run -d -p 80:80 -p 81:81 -v /mnt:/etc/nginx/conf.d/ -v /opt/yiliao:/opt/yiliao -v /opt/xiaoniao:/opt/xiaoniao nginx:latest
```

#### 11 手动将容器保存为镜像

```bash
1）：基于容器制作镜像
docker run -it centos:6.9 
###### 
yum install httpd 
yum install openssh-server 
/etc/init.d/sshd start

vi /init.sh 
#!/bin/bash 
/etc/init.d/httpd start 
/usr/sbin/sshd -D

chmod +x /init.sh

2）将容器提交为镜像 
docker commit oldboy centos6-ssh-httpd:v1

3）测试镜像功能是否可用

手动制作的镜像，传输时间长
镜像初始命令

制作一个kodexplorer网盘docker镜像。nginx + php-fpm（httpd + php）
```

#### 12 dockerfile自动构建docker镜像

类似于ansible剧本，大小积kb

手动作镜像是：大小几百M+



dockerfile支持自定义容器的初始命令

```bash
cat dockerfile		#主要组成部分： 
FROM centos:6.9  				  #基础镜像信息 	
RUN yum install openssh-server -y #制作镜像操作指令 
CMD ["/bin/bash"] 				  #容器启动时执行指令 
```

dockerfile常用指令：

```bash
FROM     		#指定基础镜像
MAINTAINER		#指定维护者信息，可以没有）
LABLE 			#描述，标签
RUN 			#在命令前面加上RUN即可
ADD 			#复制文件（会自动解压tar）
WORKDIR 		#相当于cd，设置当前工作目录
VOLUME 			#设置卷，挂载主机目录
EXPOSE 			#指定对外的端口 -P 随机端口
CMD 			#指定容器启动后的要干的事情，容易被替换

dockerfile其他指令： 
COPY 			#复制文件（不会解压）
ENV 			#环境变量
ENTRYPOINT 		#容器启动后执行的命令（无法被替换，启容器的时候指定的命令，会被当成参数）
```

- 自动制作html镜像

```bash
[root@docker01 nginx]# cat dockerfile 
FROM centos:6.9
RUN echo "192.168.37.200 mirrors.aliyun.com" >>/etc/hosts
RUN curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
RUN curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
RUN yum install nginx -y
WORKDIR /usr/share/nginx/html
ADD monitor .
CMD ["nginx","-g","daemon off;"]

docker build -t yiliao:v1 .  #最后是dockerfile的路径
```

- 构建底层的docker系统镜像

```bash
#系统文件下载地址：
https://mirrors.tuna.tsinghua.edu.cn/lxc-images/images/alpine/

[root@docker01 dockerfile]# cat dockerfile 
FROM scratch
ADD alpine.tar.gz /
CMD ["/bin/sh"]

#构建
docker build -t alpine_me:v1 .
```

- 构建提供ssh服务的镜像

```bash
[root@docker01 sshd]# cat dockerfile 
FROM centos:7
ENV version 7.4p1
RUN curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
RUN yum install initscripts openssh-server-$version -y 
RUN /usr/sbin/sshd-keygen
ADD init.sh /init.sh
ENTRYPOINT ["/bin/bash","init.sh"]

[root@docker01 sshd]# cat init.sh 
#!/bin/bash
echo $1 | passwd --stdin root
/usr/sbin/sshd -D

```

#### 13  docker镜像的分层（kvm 链接克隆，写时复制的特性）

- 镜像分层的好处：复用,节省磁盘空间，相同的内容只需加载一份到内存。
- 修改dockerfile之后，再次构建速度快

dockerfile 优化：

- 尽可能选择体积小linux，alpine
- 尽可能合并RUN指令，清理无用的文件（yum缓存，源码包）
- 修改dockerfile，把变化的内容尽可能放在dockerfile结尾
- 使用.dockerignore，减少不必要的文件ADD . /html

![1575547665159](F:\typora\1575547665159.png)

#### 14  容器间的互联（--link 是单方向的！！！）

```bash
docker run -d -it --name db01 alpine:3.9 
docker run -it --link db01:mysql alpine:3.9

`在指定link的容器上面测试ping db01，link的实质是在容器上做了host的解析`
```

使用docker运行zabbix-server

```bash
docker run --name mysql-server -t \
	-e MYSQL_DATABASE="zabbix" \
	-e MYSQL_USER="zabbix" \
	-e MYSQL_PASSWORD="zabbix_pwd" \
	-e MYSQL_ROOT_PASSWORD="root_pwd" \
	-d mysql:5.7 \
	--character-set-server=utf8 --collation server=utf8_bin

docker run --name zabbix-java-gateway -t \
	-d zabbix/zabbix-java-gateway:latest

docker run --name zabbix-server-mysql -t \
	-e DB_SERVER_HOST="mysql-server" \
	-e MYSQL_DATABASE="zabbix" \
	-e MYSQL_USER="zabbix" \
	-e MYSQL_PASSWORD="zabbix_pwd" \
	-e MYSQL_ROOT_PASSWORD="root_pwd" \
	-e ZBX_JAVAGATEWAY="zabbix-java-gateway" \
	--link mysql-server:mysql \
	--link zabbix-java-gateway:zabbix-java-gateway \
	-p 10051:10051 \
	-d zabbix/zabbix-server-mysql:latest

docker run --name zabbix-web-nginx-mysql -t \
	-e DB_SERVER_HOST="mysql-server" \
	-e MYSQL_DATABASE="zabbix" \
	-e MYSQL_USER="zabbix" \
	-e MYSQL_PASSWORD="zabbix_pwd" \
	-e MYSQL_ROOT_PASSWORD="root_pwd" \
	--link mysql-server:mysql \
	--link zabbix-server-mysql:zabbix-server \
	-p 80:80 \
	-d zabbix/zabbix-web-nginx-mysql:latest

监控报警：微信报警，alpine    
yum 安装zabbix好使
```

#### 15 docker-compose(单机版的容器编排工具)

类似于ansible剧本

- 安装

```bash
yum install -y docker-compose   #需要epel源
```

- 名称必须为docker-compose.yml或docker-compose.yaml

- 命令

```bash
docker-compose up   #启动 
docker-compose up -d #后台启动
docker-compose down #关闭
docker-compose restart 服务名称 #重启服务
docker-compose stop 服务名称 #停止服务
docker-compose start 服务名称 #开启服务

```

- 示例

```yaml
`wordpress`
version: '3'

services:
   db:
     image: mysql:5.7
     volumes:
       - /data/db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     volumes:
       - /data/web_data:/var/www/html
     ports:
       - "80:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
```

#### 16 docker registry（私有仓库）

```bash
#启动容器
docker run -d -p 5000:5000 --restart=always --name registry -v /opt/myregistry:/var/lib/registry registry

#上传镜像到私有仓库：
#1.给镜像打标签
docker tag centos6-sshd:v3 10.0.0.11:5000/centos6-sshd:v3
#2.上传镜像
docker push 10.0.0.11:5000/centos6-sshd:v3


`如果遇到报错：`
The push refers to repository [10.0.0.11:5000/centos6.9_ssh]
Get https://10.0.0.11:5000/v2/: http: server gave HTTP response to HTTPS client

`解决方法`  #多个用逗号隔开
vim /etc/docker/daemon.json
{
"insecure-registries": ["10.0.0.11:5000"]
}
systemctl restart docker
```

**1.查看镜像列表**

​	使用浏览器访问：

​	http://10.0.0.11:5000/v2/_catalog

**2.查看镜像的版本**

​	下面我已nginx为例

​	http://10.0.0.11:5000/v2/nginx/tags/list

`向官方仓库上传文件时需要登录`

```bash
docker login
```

#### 17 docker企业级镜像仓库harbor(vmware 中国团队)

下载地址：https://github.com/goharbor/harbor

***1、基础应用***

- 安装docker和docker-compose
- 下载harbor安装包
- 上传到/opt目录（不要解压到root目录）

- 修改harbor.yml的配置文件

```yaml
hostname = 10.0.0.11
harbor_admin_password = 123456
```

- 执行install.sh

1. 推送镜像

```bash
#推送镜像

`没有配置https是需要配置信任`
vim /etc/docker/daemon.json
{
"insecure-registries": ["10.0.0.11:5000"]
}

#登录
docker login ip地址 或 域名

#在项目中标记镜像：
docker tag SOURCE_IMAGE[:TAG] blog.oldqiang.com/oldqin/IMAGE[:TAG]

#推送镜像到当前项目：
docker push blog.oldqiang.com/oldqin/IMAGE[:TAG]

```

2. 拉取镜像

3. ```bash
   docker pull blog.oldqiang.com/oldqin/busybox:latest
   ```

***2、配置https***

- 下载ssl证书
- 配置harbor.yml

```yaml
[root@docker02 harbor]# cat harbor.yml 

hostname: blog.oldqiang.com

https:
  port: 443
  certificate: /opt/cert/Nginx/1_blog.oldqiang.com_bundle.crt
  private_key: /opt/cert/Nginx/2_blog.oldqiang.com.key

```

***3、将docker-registry迁移到harbo仓库***

- 新建项目

![1575792306529](F:\typora\1575792306529.png)





![1575792819280](F:\typora\1575792819280.png)





![1575792582095](F:\typora\1575792582095.png)





![1575792910711](F:\typora\1575792910711.png)





![1575793016889](F:\typora\1575793016889.png)



![1575793078170](F:\typora\1575793078170.png)

`删除镜像后需要进行垃圾清理`

![1575793163097](F:\typora\1575793163097.png)

#### 18 Docker网络类型

`docker network`  



- None：不为容器配置任何网络功能，--net=none

- Container：与另一个运行中的容器共享Network Namespace，--net=container:containerID（K8S）

```bash
--network container:容器id
```

- Host：与宿主机共享Network Namespace，--network=host 性能最高 

- Bridge：Docker设计的NAT网络模型

**1. 创建bridge网络**

```bash 
docker network create -d bridge --subnet 172.18.0.0/16 --geteway 172.18.0.1 oldqiang
```

#### 19 Docker跨主机容器之间的通信macvlan

默认一个物理网卡，只有一个物理mac地址，虚拟多个mac地址

```bash
##创建macvlan网络
docker network create --driver macvlan --subnet 10.0.0.0/24 --gateway 10.0.0.254 -o parent=eth0 macvlan_1
##设置eth0的网卡为混杂模式
ip link set eth1 promisc on
##创建使用macvlan网络的容器
docker run -it --network macvlan_1 --ip=10.0.0.200 busybox
```

#### 20 Dcoker跨主机容器通信之overlay

docker03上：

```bash
docker run -d -p 8500:8500 -h consul --name consul progrium/consul -server -bootstrap
```

设置容器的主机名

```bash
consul：kv类型的存储数据库（key:value）
docker01、02上：
vim /etc/docker/daemon.json
{
"hosts":["tcp://0.0.0.0:2376","unix:///var/run/docker.sock"],
"cluster-store": "consul://10.0.0.13:8500",
"cluster-advertise": "10.0.0.11:2376"
}

vim /etc/docker/daemon.json 
vim /usr/lib/systemd/system/docker.service
-H
systemctl daemon-reload 
systemctl restart docker

2）创建overlay网络
docker network create -d overlay --subnet 172.16.1.0/24 --gateway 172.16.1.254 ol1

3）启动容器测试
docker run -it --network ol1 --name oldboy01 busybox /bin/bash
每个容器有两块网卡,eth0实现容器间的通讯,eth1实现容器访问外网
```

#### 21 docker 监控

***1、docker  cadvisor监控 + prometheus + grafana***

```bash
`监控节点配置`

#增加node节点
上传docker_monitor_node.tar.gz

docker load -i docker_progrium_consul.tar.gz
#启动node-exporter
docker run -d   -p 9100:9100   -v "/:/host:ro,rslave"   --name=node_exporter   quay.io/prometheus/node-exporter   --path.rootfs /host

#启动cadvisor
docker run --volume=/:/rootfs:ro  --volume=/var/run:/var/run:rw --volume=/sys:/sys:ro --volume=/var/lib/docker/:/var/lib/docker:ro  --publish=8080:8080 --detach=true --name=cadvisor google/cadvisor:latest
```

- 安装 prometheus

```bash
#安装二进制包

#修改配置文件
vim prometheus.yml
  - job_name: 'cadvisor'
    static_configs:
    - targets: ['10.0.0.11:8080','10.0.0.12:8080']
  - job_name: 'node-exporter'
    static_configs:
    - targets: ['10.0.0.11:9100','10.0.0.12:9100']
    
#启动
./prometheus --help 
./prometheus  --config.file="prometheus.yml"

```

![1575806350044](F:\typora\1575806350044.png)

- 安装grafana

```bash
#下载RPM包
https://mirrors.tuna.tsinghua.edu.cn/grafana/yum/el7/

yum localinstall 
```

访问 http://10.0.0.13:3000

![1575806650106](F:\typora\1575806650106.png)

上传dashboard

![1575806803990](F:\typora\1575806803990.png)

![1575806913020](F:\typora\1575806913020.png)

***2、zabbix低级自动发现***

- 安装zabbix

```bash
#运行zabbix的容器
[root@docker01 zabbix]# cat docker-compose.yml 
version: '3'

services:
   mysql-server:
     image: mysql:5.7
     restart: always
     command: --character-set-server=utf8 --collation-server=utf8_bin
     environment:
       MYSQL_ROOT_PASSWORD: root_pwd
       MYSQL_DATABASE: zabbix
       MYSQL_USER: zabbix
       MYSQL_PASSWORD: zabbix_pwd
   
   zabbix-java-gateway:
     image: zabbix/zabbix-java-gateway:latest
     restart: always 

   zabbix-server:
     depends_on:
       - mysql-server
       - zabbix-java-gateway
     image: zabbix/zabbix-server-mysql:latest
     ports:
       - "10051:10051"
     restart: always
     environment:
       DB_SERVER_HOST: mysql-server 
       MYSQL_DATABASE: zabbix 
       MYSQL_USER: zabbix 
       MYSQL_PASSWORD: zabbix_pwd 
       MYSQL_ROOT_PASSWORD: root_pwd 
       ZBX_JAVAGATEWAY: zabbix-java-gateway 

   zabbix-web:
     depends_on:
       - mysql-server
       - zabbix-server
     image: zabbix/zabbix-web-nginx-mysql:latest
     ports:
       - "80:80"
     restart: always
     environment:
       DB_SERVER_HOST: mysql-server 
       MYSQL_DATABASE: zabbix 
       MYSQL_USER: zabbix 
       MYSQL_PASSWORD: zabbix_pwd 
       MYSQL_ROOT_PASSWORD: root_pwd 

#访问http://10.0.0.11
```

- 配置主机

![1575807438638](F:\typora\1575807438638.png)

![1575807495578](F:\typora\1575807495578.png)

- 低级自动发现

![1575807653835](F:\typora\1575807653835.png)

- 自定义监控项原型

```bash
#自带的监控项原型使用的是内置key
zabbix_agentd -p 	#查看内置key
```

克隆

![1575808115972](F:\typora\1575808115972.png)

![1575808131445](F:\typora\1575808131445.png)

![1575808170894](F:\typora\1575808170894.png)

![1575808326518](F:\typora\1575808326518.png)

- 监控网卡的mac地址

```bash
#配置key
[root@docker02 harbor]# cat /etc/zabbix/zabbix_agentd.d/user_define.conf
UserParameter=net.if.mac[*],ifconfig $1 | awk '/ether /{print $$2}'
```

克隆

![1575809205250](F:\typora\1575809205250.png)

![1575809227781](F:\typora\1575809227781.png)

- 配置自动发现规则

```bash
#在docker宿主机上编写扫描docker容器名称的脚本
[root@node1 zabbix_agentd.d]# cat /etc/zabbix/script/docker_discovery.sh 
#!/bin/bash
port=($(/usr/bin/docker ps -a|grep -v "CONTAINER ID"|awk '{print $NF}'))
printf '{\n'
printf '\t"data":[\n'
   for key in ${!port[@]}
       do
           if [[ "${#port[@]}" -gt 1 && "${key}" -ne "$((${#port[@]}-1))" ]];then
              printf '\t {\n'
              printf "\t\t\t\"{#CONTAINERNAME}\":\"${port[${key}]}\"},\n"
​
         else [[ "${key}" -eq "((${#port[@]}-1))" ]]
              printf '\t {\n'
              printf "\t\t\t\"{#CONTAINERNAME}\":\"${port[${key}]}\"}\n"
​
           fi
   done
​
              printf '\t ]\n'
              printf '}\n'
​
#脚本效果如下：
[root@node1 zabbix_agentd.d]# sh /etc/zabbix/script/docker_discovery.sh
{
    "data":[
     {
            "{#CONTAINERNAME}":"k8s_install-cni_kube-flannel-ds-amd64-4cr9d_kube-system_39d5a60d-702c-466c-97e0-85123e1a7869_0"},
     {
            "{#CONTAINERNAME}":"k8s_nginx_nginx_default_3f5ffd65-a233-48c5-973d-b43922ccea61_0"},
     {
            "{#CONTAINERNAME}":"k8s_POD_nginx_default_3f5ffd65-a233-48c5-973d-b43922ccea61_0"},
     {
            "{#CONTAINERNAME}":"k8s_kube-flannel_kube-flannel-ds-amd64-4cr9d_kube-system_39d5a60d-702c-466c-97e0-85123e1a7869_3"},
     {
            "{#CONTAINERNAME}":"k8s_kube-proxy_kube-proxy-sqhs4_kube-system_1c7fac8f-5263-4cd8-b008-485f1dd44ad1_3"},
     {
            "{#CONTAINERNAME}":"k8s_POD_kube-flannel-ds-amd64-4cr9d_kube-system_39d5a60d-702c-466c-97e0-85123e1a7869_3"},
     {
            "{#CONTAINERNAME}":"k8s_POD_kube-proxy-sqhs4_kube-system_1c7fac8f-5263-4cd8-b008-485f1dd44ad1_3"}
     ]
}
#注意事项！！！
zabbix-agent在取值的时候使用的是zabbix用户，在执行docker ps -a会没有权限执行
解决方法有两种
1：添加sudo授权
2：给docker命令加suid权限
chmod u+s /usr/bin/docker
推荐使用第二种，比较简单


#配置key
[root@docker02 harbor]# cat 
UserParameter=docker.discovery,/bin/bash /scripts/docker_discovery.sh

```

![1575809679416](F:\typora\1575809679416.png)

- 监控docker的存活状态

```bash
[root@docker02 harbor]# cat 
UserParameter=docker.status[*],/usr/bin/docker ps -a | grep -w $1 | grep -c Up

```

