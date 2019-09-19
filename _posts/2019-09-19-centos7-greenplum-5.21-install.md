---
layout: post
title:Greenplum|Greenplum
category: Greenplum
tag: [Greenplum]
---

这篇文章主要介绍Greenplum分布式版本管理与集中式管理的一些差异，总结下Greenplum常用命令作为日后的速查表，最后介绍Git进阶的一些案例。
本文分为以下几个部分：
1. Greenplum差异

#《Greenplum5-centos7安装指南》

## 一、基础软件准备

  (1)下载RPM安装包
     这里安装的GP版本为：5.5.21.2
     greenplum-db-5.5.21.2-rhel6-x86_64.rpm
     下载地址：https://network.pivotal.io/products/pivotal-gpdb
	 
  (2)安装基础软件包 
   yum install -y net-tools which openssh-clients openssh-server less zip unzip iproute.x86_64
   
## 二、机器规划与准备工作

   （1）机器规划（一个主节点，一个从节点，两个数据节点: root帐号的密码为：whistle）：
    节点类型     节点IP         节点hostname      节点描述
	master    172.16.90.151     mdw              master 主节点
	standby   172.16.90.152     smdw             standby 从节点
	segment   172.16.90.153     sdw1             数据节点1
	segment   172.16.90.154     sdw2             数据节点2
	
	（2）分别设置三台主机的hostname
	在master机器上：
	hostnamectl --static set-hostname mdw
	在standby机器上：
	hostnamectl --static set-hostname smdw
	在segment1机器上：
	hostnamectl --static set-hostname sdw1
	在segment2机器上：
	hostnamectl --static set-hostname sdw2
	
	需要在/etc/hosts中设置节点的hostname(如上)，并能ping通。
	
	（3）关闭防火墙
	systemctl stop firewalld
	systemctl mask firewalld
	systemctl stop iptables
	systemctl disable iptables
	
   （4）每台主机上创建gpadmin用户与组
	groupdel gpadmin
	userdel gpadmin
	groupadd -g 530 gpadmin
	useradd -g 530 -u 530 -m -d /home/gpadmin -s /bin/bash gpadmin
	passwd gpadmin  注：后面均按照密码为gpadmin编写
	
	（5）创建节点配置信息
	在master上创建文件all_hosts（所有主机名）和 all_segs（2台数据节点主机名）路径可以自选，这里用/home/gpadmin/nodes
	文件内容：
	文件/home/gpadmin/nodes/all_hosts:
		mdw
		smdw
		sdw1
		sdw2

	文件/home/gpadmin/nodes/all_segs:
		sdw1
		sdw2
		
## 三、配置每台机器的内核参数
   cp /etc/sysctl.conf /etc/sysctl.conf_bak
   vim /etc/sysctl.conf

--------------------------
kernel.shmmax = 500000000
kernel.shmmni = 4096
kernel.shmall = 4000000000
kernel.sem = 500 1024000 200 4096
kernel.sysrq = 1
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.msgmni = 2048
net.ipv4.tcp_syncookies = 1
net.ipv4.ip_forward = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.conf.all.arp_filter = 1
net.ipv4.ip_local_port_range = 10000 65535
net.core.netdev_max_backlog = 10000
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
vm.overcommit_memory = 2
--------------------------
  
   执行如下使内核参数生效：
   sysctl -p
   
四、配置每台机器的资源限制参数

	vim /etc/security/limits.conf

设置文件/etc/security/limits.conf的内容如下：
-------------------
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072
----------------------

## 五、安装Greenplum

1，准备greenplum数据库安装文件(master节点)
   路径：/home/greenplum/installer：
     greenplum-db-5.X.X-rhel7-x86_64.rpm

2，安装软件(master节点)

（1）安装软件包
  cd /home/greenplum/installer/
  rpm -Uvh ./greenplum-db-5.X.X-rhel7-x86_64.rpm
  默认安装路径 /usr/local/,即安装的目录为：/usr/local/greenplum-db/
 
（3）设置安装目录的用户及组权限
  chown -R gpadmin /usr/local/greenplum*（在创建gpadmin后执行）
  chgrp -R gpadmin /usr/local/greenplum*（在创建gpadmin后执行）
  
（3）查看获取环境参数
  source /usr/local/greenplum-db/greenplum_path.sh
  echo $GPHOME

 （4）拷贝master节点公钥至其他各个节点    
  cd /usr/local/greenplum-db/bin
  source /usr/local/greenplum-db/greenplum_path.sh
  gpssh-exkeys -f /home/gpadmin/nodes/all_hosts
  
---------------------------------------------------------
[root@mdw bin]# gpssh-exkeys -f /home/gpadmin/nodes/all_hosts
[STEP 1 of 5] create local ID and authorize on local host

[STEP 2 of 5] keyscan all hosts and update known_hosts file

[STEP 3 of 5] authorize current user on remote hosts
  ... send to smdw
  ***
  *** Enter password for smdw: （机器的root帐号的密码:whistle）
  ... send to sdw1
  ... send to sdw2

[STEP 4 of 5] determine common authentication file content

[STEP 5 of 5] copy authentication files to all remote hosts
  ... finished key exchange with smdw
  ... finished key exchange with sdw1
  ... finished key exchange with sdw2

[INFO] completed successfully
---------------------------------------------------------
  
 （5）运行gpseginstall工具
  cd /usr/local/greenplum-db/bin
  source /usr/local/greenplum-db/greenplum_path.sh
  gpseginstall -f /home/gpadmin/nodes/all_hosts -u gpadmin
  
---------------------------------------------------------
[root@mdw bin]# gpseginstall -f /home/gpadmin/nodes/all_hosts -u gpadmin
20190902:16:51:00:019873 gpseginstall:mdw:root-[INFO]:-Installation Info:
link_name greenplum-db
binary_path /usr/local/greenplum-db-5.21.2
binary_dir_location /usr/local
binary_dir_name greenplum-db-5.21.2
20190902:16:51:00:019873 gpseginstall:mdw:root-[INFO]:-check cluster password access
20190902:16:51:01:019873 gpseginstall:mdw:root-[INFO]:-de-duplicate hostnames
20190902:16:51:01:019873 gpseginstall:mdw:root-[INFO]:-master hostname: mdw
20190902:16:51:01:019873 gpseginstall:mdw:root-[INFO]:-check for user gpadmin on cluster
20190902:16:51:01:019873 gpseginstall:mdw:root-[INFO]:-add user gpadmin on master
20190902:16:51:02:019873 gpseginstall:mdw:root-[INFO]:-add user gpadmin on cluster
20190902:16:51:02:019873 gpseginstall:mdw:root-[INFO]:-chown -R gpadmin:gpadmin /usr/local/greenplum-db
20190902:16:51:02:019873 gpseginstall:mdw:root-[INFO]:-chown -R gpadmin:gpadmin /usr/local/greenplum-db-5.21.2
20190902:16:51:02:019873 gpseginstall:mdw:root-[INFO]:-rm -f /usr/local/greenplum-db-5.21.2.tar; rm -f /usr/local/greenplum-db-5.21.2.tar.gz
20190902:16:51:02:019873 gpseginstall:mdw:root-[INFO]:-cd /usr/local; tar cf greenplum-db-5.21.2.tar greenplum-db-5.21.2
20190902:16:51:05:019873 gpseginstall:mdw:root-[INFO]:-gzip /usr/local/greenplum-db-5.21.2.tar
20190902:16:51:41:019873 gpseginstall:mdw:root-[INFO]:-remote command: mkdir -p /usr/local
20190902:16:51:42:019873 gpseginstall:mdw:root-[INFO]:-remote command: rm -rf /usr/local/greenplum-db-5.21.2
20190902:16:51:42:019873 gpseginstall:mdw:root-[INFO]:-scp software to remote location
20190902:16:51:44:019873 gpseginstall:mdw:root-[INFO]:-remote command: gzip -f -d /usr/local/greenplum-db-5.21.2.tar.gz
20190902:16:51:51:019873 gpseginstall:mdw:root-[INFO]:-md5 check on remote location
20190902:16:51:53:019873 gpseginstall:mdw:root-[INFO]:-remote command: cd /usr/local; tar xf greenplum-db-5.21.2.tar
20190902:16:51:54:019873 gpseginstall:mdw:root-[INFO]:-remote command: rm -f /usr/local/greenplum-db-5.21.2.tar
20190902:16:51:55:019873 gpseginstall:mdw:root-[INFO]:-remote command: cd /usr/local; rm -f greenplum-db; ln -fs greenplum-db-5.21.2 greenplum-db
20190902:16:51:55:019873 gpseginstall:mdw:root-[INFO]:-remote command: chown -R gpadmin:gpadmin /usr/local/greenplum-db
20190902:16:51:56:019873 gpseginstall:mdw:root-[INFO]:-remote command: chown -R gpadmin:gpadmin /usr/local/greenplum-db-5.21.2
20190902:16:51:56:019873 gpseginstall:mdw:root-[INFO]:-rm -f /usr/local/greenplum-db-5.21.2.tar.gz
Please enter a password: (这里输入：gpadmin)
Confirm password: (这里输入：gpadmin)
20190902:16:52:05:019873 gpseginstall:mdw:root-[INFO]:-Changing system passwords ...
20190902:16:52:06:019873 gpseginstall:mdw:root-[INFO]:-exchange ssh keys for user root
20190902:16:52:09:019873 gpseginstall:mdw:root-[INFO]:-exchange ssh keys for user gpadmin
20190902:16:52:12:019873 gpseginstall:mdw:root-[INFO]:-/usr/local/greenplum-db/./sbin/gpfixuserlimts -f /etc/security/limits.conf -u gpadmin
20190902:16:52:13:019873 gpseginstall:mdw:root-[INFO]:-remote command: . /usr/local/greenplum-db/./greenplum_path.sh; /usr/local/greenplum-db/./sbin/gpfixuserlimts -f /etc/security/limits.conf -u gpadmin
20190902:16:52:13:019873 gpseginstall:mdw:root-[INFO]:-version string on master: gpssh version 5.21.2 build commit:610b6d777436fe4a281a371cae85ac40f01f4f5e
20190902:16:52:13:019873 gpseginstall:mdw:root-[INFO]:-remote command: . /usr/local/greenplum-db/./greenplum_path.sh; /usr/local/greenplum-db/./bin/gpssh --version
20190902:16:52:14:019873 gpseginstall:mdw:root-[INFO]:-remote command: . /usr/local/greenplum-db-5.21.2/greenplum_path.sh; /usr/local/greenplum-db-5.21.2/bin/gpssh --version
20190902:16:52:14:019873 gpseginstall:mdw:root-[INFO]:-SUCCESS -- Requested commands completed
-----------------------------------------------------------
至此完成其他2台主机的安装

  （6）切换到gpadmin用户验证无密码登录
	su - gpadmin
	source /usr/local/greenplum-db/greenplum_path.sh
	gpssh -f /home/gpadmin/nodes/all_hosts -e ls -l $GPHOME

  （7）配置环境变量
	在上述（6）切换到gpadmin帐号前提下：
	vim ~/.bashrc
	
---------------------------------------------------------------
	source /usr/local/greenplum-db/greenplum_path.sh
	export MASTER_DATA_DIRECTORY=/home/gpadmin/data/master/gpseg-1/  (改目录事先建好)
	export PGPORT=5432
	export PGUSER=gpadmin
	export PGDATABASE=postgres                   （默认数据库）
---------------------------------------------------------------

	使环境变量生效：source ~/.bashrc 

	（8）使用gpssh工具在所有segment主机上创建主数据和镜像数据目录，如果没有设置镜像可以不创建mirror目录（执行下面命令）:
	 gpssh -f /home/gpadmin/nodes/all_segs -e 'mkdir -p /home/gpadmin/data/gpseg'
 
	（9）在Master主机上，通过NTP守护进程同步系统时钟（略）

	（10）检测系统环境（root帐号下）
	   source /usr/local/greenplum-db/greenplum_path.sh
	   cd /home/gpadmin/
	   
	  a)Master上进行主机OS参数检测
	   gpcheck -f /home/gpadmin/nodes/all_hosts -m mdw
	   ----------------------------------------------------------------------
		gpcheck时遇到的一些报错解决：
		$ gpssh -f /home/gpadmin/nodes/all_hosts -e 'echo deadline > /sys/block/sr0/queue/scheduler'
		$ gpssh -f /home/gpadmin/nodes/all_hosts -e 'echo deadline > /sys/block/sr1/queue/scheduler'
		$ gpssh -f /home/gpadmin/nodes/all_hosts -e 'echo deadline > /sys/block/sda/queue/scheduler'
		$ /sbin/blockdev --setra 16384 /dev/sda* 
		$ /sbin/blockdev --getra /dev/sda*
	   ----------------------------------------------------------------------
	   
	   b)Master上检查网络性能
	   $ gpcheckperf -f /home/gpadmin/nodes/all_segs -r N -d /tmp > subnet1.out
	   $ cat subnet1.out
	   
	   c)Master上检查磁盘I/O和内存带宽
	   $ gpcheckperf -f /home/gpadmin/nodes/all_hosts -d /home/data/gpseg -r ds
   
## 六、初始化Greenplum数据库

1. 初始化数据库

	(1)在master节点上切换至gpadmin帐号下：
	su - gpadmin
	
    (2)从模板中拷贝一份gpinitsystem_config文件
    #cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_config /home/gpadmin/gpinitsystem_config
	
	(3)根据文件中的配置目录在三台机器上创建相应的目录：
	mkdir -p /home/gpadmin/data/master    (在mdw主机上)
	mkdir -p /home/gpadmin/data/master    (在standby主机上)
	mkdir -p /home/gpadmin/data/primary   (在sdw1主机上)
	mkdir -p /home/gpadmin/data/primary   (在sdw2主机上)
	
	(4)执行初始化
	$ gpinitsystem -c /home/gpadmin/gpinitsystem_config -h /home/gpadmin/nodes/all_segs
	待命令初始化成功后，三台机器上即已经成功启动了greenplum数据库了。

	(5)增加master的从节点：
	$ gpinitstandby -s smdw

2, 访问数据库
# psql -d postgres

	psql (8.2.15)
	Type "help" for help.
	postgres=#
	出现以上界面，恭喜你已经安装成功了。

	输入查询语句，查看是否可以执行。

	postgres=# select datname,datdba,encoding,datacl from pg_database;

	  datname  | datdba | encoding |              datacl             
	-----------+--------+----------+----------------------------------
	 postgres  |     10 |        6 |
	 template1 |     10 |        6 | {=c/gpadmin,gpadmin=CTc/gpadmin}
	 template0 |     10 |        6 | {=c/gpadmin,gpadmin=CTc/gpadmin}
	(3 rows)
	
	postgres=# \q（退出）

3  用户创建密码
	postgres =# alter role gpadmin with password 'gpadmin';
	
4，启动和停止数据库测试

 （1）启动
  $ gpstart
  
 （2）停止 
  $ gpstop
  或者：强制停止
  gpstop -M fast
	
5 配置远程登录	
  （1）pg_hba.conf配置文件 
     greenplum数据库底层封装的是 postgresql 数据库，与 pg 数据库一样，要想登录数据库，需先配置数据库白名单，即允许登录的数据库相关信息。配置文件为位于 MASTER 节点的数据目录之下的 pg_hba.conf 文件。
     vim $MASTER_DATA_DIRECTORY/pg_hba.conf
  
	该文件的记录有5个字段，代表的意义为：
	1、连接方式
	分别是：“local”使用本地unix套接字，“host”使用TCP/IP连接（包括SSL和非SSL），“hostssl”只能使用SSL TCP/IP连接，“hostnossl”不能使用SSL TCP/IP连接。
	2、数据库名
	可通过使用 all 关键字表示所有的数据库
	3、用户名
	多个用户通过逗号隔开
	4、被允许连接的服务器地址
	5、认证方式
	ident，md5，password（明文密码），trust（无需密码），reject（拒绝认证）
  （2）postgresql.conf配置文件 
     将配置文件postgresql.conf的listen_addresses修改为监听所有,也就是listen_addresses = '*'

6 建议
  （1）在master节点上通过crontab设置定期清理pg的日志
   */10 * * * * (rm -f /home/gpadmin/data/master/gpseg-1/pg_log/gpdb-*.csv)
	 
7 FAQ
	（1）20190902:15:18:30:014016 gpstart:mdw:gpadmin-[WARNING]:-FATAL:  DTM initialization: failure during startup recovery, retry failed, check segment status (cdbtm.c:1529)
只能配置segment节点，用作磁盘读写的内存缓冲区,开始可以设置一个较小的值，比如总内存的15%，然后逐渐增加，过程中监控性能提升和swap的情况。以上的缓冲区的参数为125MB，此值不易设置过大，过大或导致错误。
gpconfig -c shared_buffers -v "125MB"
	(2) 常见问题FAQ
https://blog.csdn.net/q936889811/article/details/85612046
http://www.360doc.com/content/19/0430/17/37882969_832566583.shtml

