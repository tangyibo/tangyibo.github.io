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

