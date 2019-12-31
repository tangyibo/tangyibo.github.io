---
layout: post
title: 关系数据库的事务与JDBC:| 数据库
category: 数据库
tag: [数据库]
---

这篇文章主要介绍了关系数据库的事务与JDBC相关。
本文分为以下几个部分：
1. 事务的概念
2. JDBC中的事务


# 1.事务的概念

事务(Transaction)，解决的是多个操作执行时，同时成功或同时失败的问题。比如银行转账时，扣减A的资金和增加B的资金必须一起发生，不然银行就干不下去了。

但是事务的存在，在多用户同时访问相同的数据时会产生冲突。可能出现以下几种不确定情况：

## 1.1 更新丢失

两个事务同时更新一行数据(事务都未提交)，一个事务对数据的更新把另一个事务对数据的更新覆盖了。

## 1.2 脏读

一个事务读到了另一个事务未提交的数据操作结果(未提交的数据可能会被回滚)

## 1.3 不可重复读

一个事务对同一行数据重复读取两次，但是却得到了不同的结果。包括两种情况：

- 虚读：事务1读取某数据后，事务2对其做了修改，当事务1再次读改数据时得到与前一次不同的值。
- 幻读：事务在操作过程中进行两次查询，第二次查询的结果包含或缺少了第一次查询中的数据(一般为范围查询，两次查询sql可不一样)。这是因为两次查询过程中有另一事务插入或删除数据造成的。

为了解决上述情况，标准sql规范中，定义了4个事务隔离级别，不同的隔离级别对事务的处理不同。

- (1) Read Uncommitted(未授权读取)：允许脏读取，但不允许更新丢失。如果一个事务已经开始写数据，则另外一个事务则不允许同时进行写操作，但允许其他事务读此行数据。
- (2) Read Committed(授权读取)：允许不可重复读取，但不允许脏读取。读取数据的事务允许其他事务继续访问该行数据，但是未提交的写事务将会禁止其他事务访问该行。
- (3) Repeatable Read(可重复读取)：静止不可重复读取和脏读取，但有时可能出现幻读。读取数据的事务将会禁止写事务(但允许写事务)，写事务则禁止任何其他事务。
- (4) Serializable(序列化)：提供严格的事务隔离。要求事务序列化执行，不能并发执行。

> 隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大。

# 2.JDBC中的事务

Java的JDBC(Java DataBase Connectivity)为访问数据库提供了一套抽象的API，通过执行SQL语句，可实现多种关系型数据库提供统一访问。

## 2.1 自动提交

JDBC中，一个connection被创建时，默认是auto-commit模式，也即一个sql statement作为一个事务，执行完成后自动commit。如果支持多个statement组成一个事务，则要禁止auto-commit模式：
```
con.setAutoCommit(false);
```

## 2.2 事务隔离级别

Connection支持了标准的四种隔离级别，如果没有设置，则查询数据库的默认隔离级别，也可以用setTransactionIsolation方法手动设置：

```
conn.setTransactionIsolation(Connection.TRANSACTION_SERIALIZABLE);
```

JDBC对应的四种隔离级别由低到高如下：
```
(1) Read Uncommitted(未授权读取)：Connection.TRANSACTION_READ_UNCOMMITTED;
(2) Read Committed(授权读取)：Connection.TRANSACTION_READ_COMMITTED;
(3) Repeatable Read(可重复读取): Connection.TRANSACTION_REPEATABLE_READ;
(4) Serializable(序列化): Connection.TRANSACTION_SERIALIZABLE
```

## 2.3 事务提交与回滚

Connection使用commit和rollback方法支持事务的提交和回滚：

```
try {
	conn.setAutoCommit(false);
	// -- do query and update operation
	conn.commit();
} catch (Exception e) {
	conn.rollback();
}
```

## 2.4 保存点的设置和释放

JDBC使用setSavepoint和releaseSavepoint方法支持保存点的设置和释放。

```
try {
	conn.setAutoCommit(false);
	// -- update 1
	Savepoint s1 = conn.setSavepoint();
	try {
		// -- update2
	} catch (Exception e) {
		conn.releaseSavepoint(s1);
	}
	conn.commit();
} catch (Exception e) {
	conn.rollback();
}
```

# 3.DataSource接口

## 3.1 DataSource介绍

DataSource表示一种创建Connection的工厂，在jdk 1.4引入，相对DriverManager的方式更优先推荐使用DataSource。支持三种实现类型：

- 基本实现：产生一个标准连接对象
- 连接池实现：将连接对象池化处理，由一个连接池管理中间件支持
- 分布式事务实现：支持分布式事务，通常也是池化的，由一个事务管理中间件支持。

## 3.1 数据库连接池框架

基于DataSource产生了两个非常常用的数据库连接池框架：DBCP和C3P0，解决了数据库连接的复用问题，极大地提高了数据库连接的使用性能。看一个DBCP的简单用例：

```
BasicDataSource basicDataSource = new BasicDataSource();
basicDataSource.setDriverClassName("com.mysql.jdbc.Driver");
basicDataSource.setUrl("jdbc:mysql://localhost:3306/lichao");
basicDataSource.setUsername("root");
basicDataSource.setPassword("root");
basicDataSource.setInitialSize(5);  // 初始化连接数
basicDataSource.setMaxActive(30);  // 最大连接数
basicDataSource.setMaxIdle(5);  // 最大空闲连接数
basicDataSource.setMinIdle(2);  // 最小空闲连接数
Connection conn = basicDataSource.getConnection();
// sql operation ...........
// .........................
conn.close();
```

# 4.Apache的DbUtils

## 4.1 maven配置

DBUtils简化了JDBC的开发步骤，使得我们可以用更少量的代码实现连接数据库的功能，pom中配置如下：

```
<!-- https://mvnrepository.com/artifact/commons-dbutils/commons-dbutils -->
<dependency>
    <groupId>commons-dbutils</groupId>
    <artifactId>commons-dbutils</artifactId>
    <version>1.6</version>
</dependency>
```

## 4.2 DbUtils核心功能

DbUtils三个核心功能包括：

- QueryRunner中提供对sql语句操作的API；
- ResultSetHandler接口，用于定义select操作后，怎样封装结果集；
- DbUtils类是一个工具类，定义了关闭资源与事务处理的方法；

