---
layout: post
title: PHP扩展pgsql和pdo_pgsql安装| php
category: php
tag: [php]
---

这篇文章主要介绍CentOS环境下安装配置PHP扩展pgsql和pdo_pgsql的方法。
本文分为以下几个部分：
1. 下载PHP7源代码
2. 安装PG的开发包
3. 编译pgsql扩展
4. 编译pdo_pgsql扩展
5. 配置PHP扩展

# PHP扩展pgsql和pdo_pgsql安装

## 一、下载PHP7源代码

···
wget https://www.php.net/distributions/php-7.2.25.tar.gz
tar zxvf php-7.2.25.tar.gz
cd 
···

## 二、安装PG的开发包

```
yum install postgresql-devel
```

以下假设PHP的安装目录为：/usr/local/whistle/php

## 三、编译pgsql扩展

```
php-7.2.25/ext/pgsql/
/usr/local/bin/phpize
./configure --with-php-config=/usr/local/whistle/php/bin/php-config
make
```

编译完成后会在php-7.2.25/ext/pgsql/modules目录下生成pgsql.so

## 四、编译pdo_pgsql扩展

```
php-7.2.25/ext/pdo_pgsql/
/usr/local/bin/phpize
./configure --with-php-config=/usr/local/whistle/php/bin/php-config
make
```

编译完成后会在php-7.2.25/ext/pdo_pgsql/modules目录下生成pdo_pgsql.so

## 五、配置PHP扩展

将上述生成的pgsql.so和pdo_pgsql.sql放到/usr/local/whistle/php/ext/目录下，然后
在php.ini文件中增加如下内容：

```
extension_dir = "/usr/local/whistle/php/ext"
extension = "pgsql.so"
extension = "pdo_pgsql.so"

```

重启PHP服务

```
service php-fpm restart
```

然后执行如下命令查看是否已经成功安装这两个扩展模块：

```
/usr/local/whistle/php/bin/php -m
```

## 六、扩展内容之——安装 PHP的kafka扩展

### 1、安装librdkafka前需要安装sasl开发包

```
yum install cyrus-sasl-lib cyrus-sasl-devel libgsasl-devel
```

### 2、安装librdkafka

先到https://github.com/edenhill/librdkafka上下载librdkafka源码包，并安装如下命令编译：

```
cd librdkafka/
./configure
make
make install
```

### 3、安装php-rdkafka

```
cd php-7.2.25/ext/rdkafka
phpize
./configure
make all -j 5
```

### 4、配置PHP扩展

将module目录下的rdkafka.so文件拷贝的php的ext目录下，并增加php.inp中的扩展模块配置重启php即可。

## 七、扩展内容之——安装 PHP的Oracle驱动OCI扩展

### 1、下载Oralce的驱动
```
	instantclient_12_2
```

### 2、编译OCI扩展
```
cd php-7.2.25/ext/pdo_oci
phpize
./configure --with-pdo-oci=instantclient,/usr/local/whistle/instantclient_12_2,12.2
make
```

编译成功后会在ext/pdo_oci/modules目录下生成：pdo_oci.so 文件

### 3、配置PHP扩展

将目录/usr/local/whistle/instantclient_12_2和将module目录下pdo_oci.so配置到PHP环境下即可。

pdo_oci.so使用绝对路径依赖oracle的驱动库:
```
[root@localhost ext]# ldd pdo_oci.so 
	linux-vdso.so.1 =>  (0x00007ffcb7599000)
	libclntsh.so.12.1 => /usr/local/whistle/instantclient_12_2/libclntsh.so.12.1 (0x00007f9e3b5cc000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f9e3b1fe000)
	libmql1.so => /usr/local/whistle/instantclient_12_2/libmql1.so (0x00007f9e3af87000)
	libipc1.so => /usr/local/whistle/instantclient_12_2/libipc1.so (0x00007f9e3ab54000)
	libnnz12.so => /usr/local/whistle/instantclient_12_2/libnnz12.so (0x00007f9e3a40b000)
	libons.so => /usr/local/whistle/instantclient_12_2/libons.so (0x00007f9e3a1bd000)
	libdl.so.2 => /lib64/libdl.so.2 (0x00007f9e39fb9000)
	libm.so.6 => /lib64/libm.so.6 (0x00007f9e39cb7000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f9e39a9b000)
	libnsl.so.1 => /lib64/libnsl.so.1 (0x00007f9e39881000)
	librt.so.1 => /lib64/librt.so.1 (0x00007f9e39679000)
	libaio.so.1 => /lib64/libaio.so.1 (0x00007f9e39477000)
	libresolv.so.2 => /lib64/libresolv.so.2 (0x00007f9e3925e000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f9e3f279000)
	libclntshcore.so.12.1 => /usr/local/whistle/instantclient_12_2/libclntshcore.so.12.1 (0x00007f9e38c90000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007f9e38a7a000)
```