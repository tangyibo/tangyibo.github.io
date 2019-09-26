---
layout: post
title: Oracle安装|Oracle数据库 
category: oracle
tag: [oracle]
---

这篇文章主要介绍通过docker安装oracle。
本文分为以下几个部分：
1. 安装docker并启动docker服务
2. 拉取 docker 镜像
3. 启动与配置Oralce环境

## CentOS7下通过docker安装oracle

> 参考教程：https://blog.csdn.net/qq_38380025/article/details/80647620

### 1、安装docker并启动docker服务
```
 yum install docker
 systemctl start docker
```
### 2 、拉取 docker 镜像（镜像6.8G）
```
 docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```
 查看本地存在的所有镜像：
```
 docker images
```
### 3、运行镜像为名称为oracle的容器
```
  docker run -d -p 1521:1521 --name oracle registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```
#### 3.1、检查容器是否运行成功
[root@localhost ~]# docker ps -a

#### 3.2、启动oracle
[root@localhost ~]# docker start oracle

#### 3.2、登录oracle容器
docker  exec -it oracle /bin/bash
[oracle@e618df208665 /]$

#### 3.3、切换到root 用户下
  su root
  密码：helowin
[oracle@e618df208665 /]$ su root
[root@e618df208665 /]$

#### 3.4、编辑/etc/profile文件配置ORACLE环境变量
```
export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
export ORACLE_SID=helowin
export PATH=$ORACLE_HOME/bin:$PATH
```
#### 3.5、创建软连接
```
ln -s $ORACLE_HOME/bin/sqlplus /usr/bin
```
#### 3.5、使用sqlplus登录数据库
su - oracle  ## 切换到oralce帐号下
```
[root@e618df208665 ~]# sqlplus /nolog

SQL*Plus: Release 11.2.0.1.0 Production on Thu Sep 26 09:09:03 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.

SQL> 

----------------------------------------
SQL> conn / as  sysdba                                ## 使用sysdba 连接oracle，最大权限，os认证，只能在本机上登陆使用。
Connected.
SQL> alter user system identified by system;          ## 修改用户 system 的密码为 system ，可以自定义
User altered.
SQL> alter user sys identified by sys;                ## 修改用户 sys 的密码为 sys ，可以自定义
User altered.
SQL> exit      ##退出编辑SQL
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
----------------------------------------

[oracle@e618df208665 /]$ exit    ##回到root用户
```

### 4、提交修改
使用docker commit 容器名称或ID 新的镜像名称:版本提交修改。
```
docker commit oracle
```
### 5、镜像的导出
使用save命令导出容器为镜像
```
docker save [options] images [images...] 
docker save -o ./centos-6.5-oralce-11g-tang.tar 8e9db3e4d7a1
```
### 6、停止容器
使用docker stop $CONTAINER_ID来终止一个运行中的容器

### 7、镜像的导入与使用
#### 7.1、使用load命令导入镜像
docker load [options] 
```
docker load < ./centos-6.5-oralce-11g-tang.tar
```
#### 7.2、镜像修改tag
```
docker tag 8e9db3e4d7a1 centos-6.5-oralce-11g-tang:v0.0.1
```
#### 7.3、运行镜像为名称为oracle的容器
```
docker run -d -p 1521:1521 --name oracle centos-6.5-oralce-11g-tang:v0.0.1
```
#### 7.4、登录oracle容器
```
docker  exec -it oracle /bin/bash
```