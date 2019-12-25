---
layout: post
title: 常用关系数据库Oracle、MySQL、PostgreSQL的SQL语法的差异整理:| 数据库
category: 数据库
tag: [数据库]
---

这篇文章主要总结了常用关系数据库Oracle、MySQL、PostgreSQL的SQL语法的差异。
本文分为以下几个部分：
1. SQL结尾的分号问题
2. SQL中的引号问题
3. 批量插入insert问题
4. 分页查询问题

## 1、SQL结尾的分号问题

- MySQL数据库的SQL允许使用英文分号结尾

- Oracle数据库的SQL不允许使用英文分号结尾

- PostgreSQL数据库的SQL不允许使用英文分号结尾

## 2、SQL中的引号问题

- MySQL数据库的SQL中使用单撇号`

- Oracle数据库的SQL中使用双引号"

- PostgreSQL数据库的SQL中使用双引号"

## 3、批量插入insert问题

- MySQL数据库的SQL中insert 支持多个values批量插入数据

```
-- ----------------------------
-- Table structure for stu
-- ----------------------------
DROP TABLE IF EXISTS `stu`;
CREATE TABLE `stu` (
  `id` varchar(128) NOT NULL,
  `name` varchar(255) NOT NULL,
  `sex` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of stu
-- ----------------------------
INSERT INTO `stu`(`id`,`name`,`sex`) VALUES 
('tangqi', '唐七', 'girl'),
('wangwu', '王五', 'boy'),
('zhangsan', '张三', 'boy');
```

- Oracle数据库的SQL中insert 支持多个values批量插入数据

```
-- ----------------------------
-- Table structure for stu
-- ----------------------------
DROP TABLE "stu";
CREATE TABLE "stu" (
  "id" varchar2(128) NOT NULL,
  "name" varchar2(255) NOT NULL,
  "sex" varchar2(255) DEFAULT NULL,
  PRIMARY KEY ("id")
) 

-- ----------------------------
-- Records of stu
-- ----------------------------
INSERT ALL
 INTO "stu"("id","name","sex") VALUES ('tangqi', '唐七', 'girl')
 INTO "stu"("id","name","sex") VALUES ('wangwu', '王五', 'boy')
 INTO "stu"("id","name","sex") VALUES ('zhangsan', '张三', 'boy')
SELECT * FROM dual 
```

> 注意：oracle中drop表时不支持IF EXISTS关键词

- PostgreSQL数据库的SQL中insert 支持多个values批量插入数据

```
-- ----------------------------
-- Table structure for stu
-- ----------------------------
DROP TABLE "stu";
CREATE TABLE "stu" (
  "id" varchar(128) NOT NULL,
  "name" varchar(255) NOT NULL,
  "sex" varchar(255) DEFAULT NULL,
  PRIMARY KEY ("id")
) 

-- ----------------------------
-- Records of stu
-- ----------------------------
INSERT INTO "stu"("id","name","sex") VALUES 
('tangqi', '唐七', 'girl'),
('wangwu', '王五', 'boy'),
('zhangsan', '张三', 'boy')
```

> 总结：PostgreSQL与MySQL的批量insert语句很相似

## 4、分页查询问题

- MySQL数据库的分页查询SQL

使用LIMIT 关键词分页:

```
SELECT * from `stu` LIMIT 0,1;

SELECT * from `stu` LIMIT 1;
```

- Oracle数据库的分页查询SQL

(1) 在Oracle11g及其一下版本使用子查询的方式分页：

```
SELECT
	*
FROM
	(
		SELECT
			ROWNUM AS rowno,
			T.*
		FROM
			"stu"  T
		where ROWNUM <2
	) table_alias
WHERE
	table_alias.rowno >= 1
```

(2) 在Oracle12c及其以上版本支持使用FETCH NEXT / ROWS ONLY 关键词方式来分页。

- PostgreSQL数据库的分页查询SQL

使用标准的LIMIT OFFSET关键词分页:

```
SELECT * from "stu" LIMIT 1 OFFSET 0
```

## 5、自增字段问题

- MySQL数据库的自增字段

（1）在建表时，对于整型字段，可以使用AUTO_INCREMENT关键词配置为自增字段；

（2）MySQL数据库中的一个表最多有一个自增的字段

- Oracle数据库的自增字段

与PostgreSQL类似.

- PostgreSQL数据库的自增字段

（1）创建表时，如果使用serial类型，默认生成的自增序列名为：'表名' + '_' + '字段名' + '_' + 'seq'；

（2）可以单纯创建序列，在建表语句中自增的字段指定使用的序列名称。示例如下：

```
CREATE SEQUENCE gys.mytable_myid_seq
    INCREMENT 1
    START 1
    MINVALUE 1
    MAXVALUE 99999999
    CACHE 1;
```
 
