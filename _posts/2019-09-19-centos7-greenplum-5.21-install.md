---
layout: post
title:Greenplum|Greenplum
category: Greenplum
tag: [Greenplum]
---

��ƪ������Ҫ����Greenplum�ֲ�ʽ�汾�����뼯��ʽ�����һЩ���죬�ܽ���Greenplum����������Ϊ�պ���ٲ��������Git���׵�һЩ������
���ķ�Ϊ���¼������֣�
1. Greenplum����


#��Greenplum5-centos7��װָ�ϡ�

## һ���������׼��

  (1)����RPM��װ��
     ���ﰲװ��GP�汾Ϊ��5.5.21.2
     greenplum-db-5.5.21.2-rhel6-x86_64.rpm
     ���ص�ַ��https://network.pivotal.io/products/pivotal-gpdb
	 
  (2)��װ��������� 
   yum install -y net-tools which openssh-clients openssh-server less zip unzip iproute.x86_64
   
## ���������滮��׼������

   ��1�������滮��һ�����ڵ㣬һ���ӽڵ㣬�������ݽڵ�: root�ʺŵ�����Ϊ��whistle����
    �ڵ�����     �ڵ�IP         �ڵ�hostname      �ڵ�����
	master    172.16.90.151     mdw              master ���ڵ�
	standby   172.16.90.152     smdw             standby �ӽڵ�
	segment   172.16.90.153     sdw1             ���ݽڵ�1
	segment   172.16.90.154     sdw2             ���ݽڵ�2
	
	��2���ֱ�������̨������hostname
	��master�����ϣ�
	hostnamectl --static set-hostname mdw
	��standby�����ϣ�
	hostnamectl --static set-hostname smdw
	��segment1�����ϣ�
	hostnamectl --static set-hostname sdw1
	��segment2�����ϣ�
	hostnamectl --static set-hostname sdw2
	
	��Ҫ��/etc/hosts�����ýڵ��hostname(����)������pingͨ��
	
	��3���رշ���ǽ
	systemctl stop firewalld
	systemctl mask firewalld
	systemctl stop iptables
	systemctl disable iptables
	
   ��4��ÿ̨�����ϴ���gpadmin�û�����
	groupdel gpadmin
	userdel gpadmin
	groupadd -g 530 gpadmin
	useradd -g 530 -u 530 -m -d /home/gpadmin -s /bin/bash gpadmin
	passwd gpadmin  ע���������������Ϊgpadmin��д
	
	��5�������ڵ�������Ϣ
	��master�ϴ����ļ�all_hosts���������������� all_segs��2̨���ݽڵ���������·��������ѡ��������/home/gpadmin/nodes
	�ļ����ݣ�
	�ļ�/home/gpadmin/nodes/all_hosts:
		mdw
		smdw
		sdw1
		sdw2

	�ļ�/home/gpadmin/nodes/all_segs:
		sdw1
		sdw2
		
## ��������ÿ̨�������ں˲���
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
  
   ִ������ʹ�ں˲�����Ч��
   sysctl -p
   
�ġ�����ÿ̨��������Դ���Ʋ���

	vim /etc/security/limits.conf

�����ļ�/etc/security/limits.conf���������£�
-------------------
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072
----------------------

## �塢��װGreenplum

1��׼��greenplum���ݿⰲװ�ļ�(master�ڵ�)
   ·����/home/greenplum/installer��
     greenplum-db-5.X.X-rhel7-x86_64.rpm

2����װ���(master�ڵ�)

��1����װ�����
  cd /home/greenplum/installer/
  rpm -Uvh ./greenplum-db-5.X.X-rhel7-x86_64.rpm
  Ĭ�ϰ�װ·�� /usr/local/,����װ��Ŀ¼Ϊ��/usr/local/greenplum-db/
 
��3�����ð�װĿ¼���û�����Ȩ��
  chown -R gpadmin /usr/local/greenplum*���ڴ���gpadmin��ִ�У�
  chgrp -R gpadmin /usr/local/greenplum*���ڴ���gpadmin��ִ�У�
  
��3���鿴��ȡ��������
  source /usr/local/greenplum-db/greenplum_path.sh
  echo $GPHOME

 ��4������master�ڵ㹫Կ�����������ڵ�    
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
  *** Enter password for smdw: ��������root�ʺŵ�����:whistle��
  ... send to sdw1
  ... send to sdw2

[STEP 4 of 5] determine common authentication file content

[STEP 5 of 5] copy authentication files to all remote hosts
  ... finished key exchange with smdw
  ... finished key exchange with sdw1
  ... finished key exchange with sdw2

[INFO] completed successfully
---------------------------------------------------------
  
 ��5������gpseginstall����
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
Please enter a password: (�������룺gpadmin)
Confirm password: (�������룺gpadmin)
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
�����������2̨�����İ�װ

  ��6���л���gpadmin�û���֤�������¼
	su - gpadmin
	source /usr/local/greenplum-db/greenplum_path.sh
	gpssh -f /home/gpadmin/nodes/all_hosts -e ls -l $GPHOME

  ��7�����û�������
	��������6���л���gpadmin�ʺ�ǰ���£�
	vim ~/.bashrc
	
---------------------------------------------------------------
	source /usr/local/greenplum-db/greenplum_path.sh
	export MASTER_DATA_DIRECTORY=/home/gpadmin/data/master/gpseg-1/  (��Ŀ¼���Ƚ���)
	export PGPORT=5432
	export PGUSER=gpadmin
	export PGDATABASE=postgres                   ��Ĭ�����ݿ⣩
---------------------------------------------------------------

	ʹ����������Ч��source ~/.bashrc 

	��8��ʹ��gpssh����������segment�����ϴ��������ݺ;�������Ŀ¼�����û�����þ�����Բ�����mirrorĿ¼��ִ���������:
	 gpssh -f /home/gpadmin/nodes/all_segs -e 'mkdir -p /home/gpadmin/data/gpseg'
 
	��9����Master�����ϣ�ͨ��NTP�ػ�����ͬ��ϵͳʱ�ӣ��ԣ�

	��10�����ϵͳ������root�ʺ��£�
	   source /usr/local/greenplum-db/greenplum_path.sh
	   cd /home/gpadmin/
	   
	  a)Master�Ͻ�������OS�������
	   gpcheck -f /home/gpadmin/nodes/all_hosts -m mdw
	   ----------------------------------------------------------------------
		gpcheckʱ������һЩ��������
		$ gpssh -f /home/gpadmin/nodes/all_hosts -e 'echo deadline > /sys/block/sr0/queue/scheduler'
		$ gpssh -f /home/gpadmin/nodes/all_hosts -e 'echo deadline > /sys/block/sr1/queue/scheduler'
		$ gpssh -f /home/gpadmin/nodes/all_hosts -e 'echo deadline > /sys/block/sda/queue/scheduler'
		$ /sbin/blockdev --setra 16384 /dev/sda* 
		$ /sbin/blockdev --getra /dev/sda*
	   ----------------------------------------------------------------------
	   
	   b)Master�ϼ����������
	   $ gpcheckperf -f /home/gpadmin/nodes/all_segs -r N -d /tmp > subnet1.out
	   $ cat subnet1.out
	   
	   c)Master�ϼ�����I/O���ڴ����
	   $ gpcheckperf -f /home/gpadmin/nodes/all_hosts -d /home/data/gpseg -r ds
   
## ������ʼ��Greenplum���ݿ�

1. ��ʼ�����ݿ�

	(1)��master�ڵ����л���gpadmin�ʺ��£�
	su - gpadmin
	
    (2)��ģ���п���һ��gpinitsystem_config�ļ�
    #cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_config /home/gpadmin/gpinitsystem_config
	
	(3)�����ļ��е�����Ŀ¼����̨�����ϴ�����Ӧ��Ŀ¼��
	mkdir -p /home/gpadmin/data/master    (��mdw������)
	mkdir -p /home/gpadmin/data/master    (��standby������)
	mkdir -p /home/gpadmin/data/primary   (��sdw1������)
	mkdir -p /home/gpadmin/data/primary   (��sdw2������)
	
	(4)ִ�г�ʼ��
	$ gpinitsystem -c /home/gpadmin/gpinitsystem_config -h /home/gpadmin/nodes/all_segs
	�������ʼ���ɹ�����̨�����ϼ��Ѿ��ɹ�������greenplum���ݿ��ˡ�

	(5)����master�Ĵӽڵ㣺
	$ gpinitstandby -s smdw

2, �������ݿ�
# psql -d postgres

	psql (8.2.15)
	Type "help" for help.
	postgres=#
	�������Ͻ��棬��ϲ���Ѿ���װ�ɹ��ˡ�

	�����ѯ��䣬�鿴�Ƿ����ִ�С�

	postgres=# select datname,datdba,encoding,datacl from pg_database;

	  datname  | datdba | encoding |              datacl             
	-----------+--------+----------+----------------------------------
	 postgres  |     10 |        6 |
	 template1 |     10 |        6 | {=c/gpadmin,gpadmin=CTc/gpadmin}
	 template0 |     10 |        6 | {=c/gpadmin,gpadmin=CTc/gpadmin}
	(3 rows)
	
	postgres=# \q���˳���

3  �û���������
	postgres =# alter role gpadmin with password 'gpadmin';
	
4��������ֹͣ���ݿ����

 ��1������
  $ gpstart
  
 ��2��ֹͣ 
  $ gpstop
  ���ߣ�ǿ��ֹͣ
  gpstop -M fast
	
5 ����Զ�̵�¼	
  ��1��pg_hba.conf�����ļ� 
     greenplum���ݿ�ײ��װ���� postgresql ���ݿ⣬�� pg ���ݿ�һ����Ҫ���¼���ݿ⣬�����������ݿ���������������¼�����ݿ������Ϣ�������ļ�Ϊλ�� MASTER �ڵ������Ŀ¼֮�µ� pg_hba.conf �ļ���
     vim $MASTER_DATA_DIRECTORY/pg_hba.conf
  
	���ļ��ļ�¼��5���ֶΣ����������Ϊ��
	1�����ӷ�ʽ
	�ֱ��ǣ���local��ʹ�ñ���unix�׽��֣���host��ʹ��TCP/IP���ӣ�����SSL�ͷ�SSL������hostssl��ֻ��ʹ��SSL TCP/IP���ӣ���hostnossl������ʹ��SSL TCP/IP���ӡ�
	2�����ݿ���
	��ͨ��ʹ�� all �ؼ��ֱ�ʾ���е����ݿ�
	3���û���
	����û�ͨ�����Ÿ���
	4�����������ӵķ�������ַ
	5����֤��ʽ
	ident��md5��password���������룩��trust���������룩��reject���ܾ���֤��
  ��2��postgresql.conf�����ļ� 
     �������ļ�postgresql.conf��listen_addresses�޸�Ϊ��������,Ҳ����listen_addresses = '*'

6 ����
  ��1����master�ڵ���ͨ��crontab���ö�������pg����־
   */10 * * * * (rm -f /home/gpadmin/data/master/gpseg-1/pg_log/gpdb-*.csv)
	 
7 FAQ
	��1��20190902:15:18:30:014016 gpstart:mdw:gpadmin-[WARNING]:-FATAL:  DTM initialization: failure during startup recovery, retry failed, check segment status (cdbtm.c:1529)
ֻ������segment�ڵ㣬�������̶�д���ڴ滺����,��ʼ��������һ����С��ֵ���������ڴ��15%��Ȼ�������ӣ������м������������swap����������ϵĻ������Ĳ���Ϊ125MB����ֵ�������ù��󣬹�����´���
gpconfig -c shared_buffers -v "125MB"
	(2) ��������FAQ
https://blog.csdn.net/q936889811/article/details/85612046
http://www.360doc.com/content/19/0430/17/37882969_832566583.shtml

