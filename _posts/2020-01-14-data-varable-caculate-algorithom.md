---
layout: post
title: 基于同一张表前后两次的全量数据计算变化量:| 算法
category: 算法
tag: [算法]
---

这篇文章主要介绍了基于同一张表前后两次的全量数据计算变化量的方法。


# 一、说在前面的话
一些涉及数据分析处理的系统，常常需要将先将业务系统中关系数据库内的数据（离线）抽取到自己的数据库中（当前比较流行的开源MPP数据库如Greenplum）以便进行后续处理，鉴于每次进行全量数据抽取，全量分析处理代价较大，需要计算同一张表前后两次的全量数据计算变化量，这种变化量数据包括insert、update、delete等，后续分析处理只针对这些变化量数据进行，由于业务系统中变化的数据要远远少于全量数据，那么只处理变化的数据将会大大加速数据处理的速度。

同步发布的博客地址：https://blog.csdn.net/inrgihc/article/details/103932231

# 二、已有的方法分析

- 触发器
在要抽取的表上建立需要的触发器，一般要建立插入、修改、删除三个触发器，每当源表中的数据发生变化，就被相应的触发器将变化的数据写入一个临时表，抽取线程从临时表中抽取数据，临时表中抽取过的数据被标记或删除。触发器方式的优点是数据抽取的性能较高，缺点是要求业务表建立触发器，对业务系统有一定的影响。

- 时间戳
它是一种基于快照比较的变化数据捕获方式，在源表上增加一个时间戳字段，系统中更新修改表数据的时候，同时修改时间戳字段的值。

当进行数据抽取时，通过比较系统时间与时间戳字段的值来决定抽取哪些数据。有的数据库的时间戳支持自动更新，即表的其它字段的数据发生改变时，自动更新时间戳字段的值。

有的数据库不支持时间戳的自动更新，这就要求业务系统在更新业务数据时，手工更新时间戳字段。同触发器方式一样，时间戳方式的性能也比较好，数据抽取相对清楚简单，但对业务系统也有很大的倾入性（加入额外的时间戳字段），特别是对不支持时间戳的自动更新的数据库，还要求业务系统进行额外的更新时间戳操作。另外，无法捕获对时间戳以前数据的delete和update操作,在数据准确性上受到了一定的限制。

- 日志分析
通过分析数据库自身的日志来判断变化的数据,例如MySQL数据库的binlog，可通过使用阿里开源的canal工具接收并将binlog推送至kafka中。

- 全表比对
典型的全表比对的方式是采用MD5校验码。

抽取工具事先为要抽取的表建立一个结构类似的MD5临时表，该临时表记录源表主键以及根据所有字段的数据计算出来的MD5校验码。每次进行数据抽取时，对源表和MD5临时表进行MD5校验码的比对，从而决定源表中的数据是新增、修改还是删除，同时更新MD5校验码。MD5方式的优点是对源系统的倾入性较小（仅需要建立一个MD5临时表），但缺点也是显而易见的，与触发器和时间戳方式中的主动通知不同，MD5方式是被动的进行全表数据的比对，当数据量大时可能存在碰撞冲突。

# 三、Kettle的变化量计算
最近在研究了kettle这种流行的ETL工具在数据变化量上的计算方法，该方法为典型的全表比对计算方法。

## 1、合并记录组件

Kettle有一个叫"合并记录"（MergeRows）的组件，该组件用于将两个不同来源的数据合并，这两个来源的数据分别为同一张业务数据表的旧数据和新数据，将旧数据和新数据按照指定的关键字匹配、比较、合并。合并后的数据将包括旧数据来源和新数据来源里的所有数据，对于变化的数据，使用新数据代替旧数据，同时在结果里用一个标示字段存储变化状态，其中几个重要的输入信息包括：

标志字段：设置标志字段的名称，标志字段用于保存比较的结果，比较结果有下列几种：

```
“identical” – 旧数据和新数据一样
“changed” – 数据发生了变化;
“new” – 新数据中有而旧数据中没有的记录
“deleted” –旧数据中有而新数据中没有的记录
```

关键字段：用于定位两个数据源中的同一条记录。

比较字段：对于两个数据源中的同一条记录中，指定需要比较的字段。

 
## 2、注意事项

在使用kettle的这个合并记录组件时需要注意一下几点：

（1）旧数据和新数据需要事先按照关键字段（即主键字段）排序。

（2）旧数据和新数据要有相同的字段名称（新旧数据表结构一致）。

## 3、隐含条件

对于MySQL等常用的关系数据库来说，有主键的表采用默认的：

          select t.a,t.b,t.c,t.d from table_name t

的SQL模式的查询结果其实是按照主键递增的顺序排序；

所以上述的“旧数据和新数据需要事先按照关键字段（即主键字段）排序”往往可以忽略，但是对于使用Greenplum这种MPP分布式数据库时，这种SQL的查询结果并不是排好序的。

# 四、变化量计算原理
通过阅读和分析kettle的MergeRow合并记录组件的源码，现将核心原理整理如下：

## 1、样本数据准备

假设业务表名称为stu，表stu_old为上一次数据抽取的全量快照数据，表stu_new为本次数据抽取的全量快照数据，表stu_diff为根据stu_old变化到stu_new数据时的变化量存储结果表，存储结果表中不仅要包含stu_old/stu_new(这两个表结构一样，都含有主键)的所有字段，还应包含有一个标记字段，stu_diff表中将标记字段取名为diff。具体定义如下：
```
-- ----------------------------

-- Table structure for stu_old

-- ----------------------------

DROP TABLE IF EXISTS `stu_old`;

CREATE TABLE `stu_old` (

  `id` varchar(128) NOT NULL,

  `name` varchar(255) NOT NULL,

  `sex` varchar(255) DEFAULT NULL,

  PRIMARY KEY (`id`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

 

-- ----------------------------

-- Records of stu_old

-- ----------------------------

INSERT INTO `stu_old` VALUES ('tangqi', '唐七', 'girl');

INSERT INTO `stu_old` VALUES ('wangwu', '王五', 'boy');

INSERT INTO `stu_old` VALUES ('zhangsan', '张三', 'boy');

 

-- ----------------------------

-- Table structure for stu_new

-- ----------------------------

DROP TABLE IF EXISTS `stu_new`;

CREATE TABLE `stu_new` (

  `id` varchar(128) NOT NULL,

  `name` varchar(255) NOT NULL,

  `sex` varchar(255) DEFAULT NULL,

  PRIMARY KEY (`id`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8;

 

-- ----------------------------

-- Records of stu_new

-- ----------------------------

INSERT INTO `stu_new` VALUES ('liliu', '刘六', 'girl');

INSERT INTO `stu_new` VALUES ('tangqi', '唐七', 'boy');

INSERT INTO `stu_new` VALUES ('wangwu', '王五', 'boy');

INSERT INTO `stu_new` VALUES ('zhangsan', '张三', 'girl');

 

-- ----------------------------

-- Table structure for stu_diff

-- ----------------------------

DROP TABLE IF EXISTS `stu_diff`;

CREATE TABLE `stu_diff` (

  `id` varchar(128) NOT NULL,

  `name` varchar(255) NOT NULL,

  `sex` varchar(255) DEFAULT NULL,

  `diff` varchar(255) NOT NULL,

  `num` int(11) NOT NULL AUTO_INCREMENT,

  PRIMARY KEY (`num`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

## 2、变化量计算过程

这里以在MySQL数据库计算为例，其中的计算过程如下：

（1）首先按照主键升序查询新旧两个表的数据：

旧表：SELECT `id`,`name`,`sex` FROM `stu_old` ORDER BY `id` asc

新表：SELECT `id`,`name`,`sex` FROM `stu_new` ORDER BY `id` asc

说明：这里使用了数据库的查询排序功能，其实也可以不用order by来排序，因为MySQL数据库有主键表的默认查询结果就是按照主键升序排序的了；但是数据库如果使用的是Greenplum时带上order by就是必要的了。

（2）结果集比较算法

比较计算的Java伪代码如下：
```
		String flagField = null;  //用于记录比较的结果状态：insert/update/delete
		Object[] outputRow;   //用于记录数据结果
		Object[] one = getRowData(rsOld.resultset);   //从旧表的结果集中取一条记录
		Object[] two = getRowData(rsNew.resultset);  //从新表的结果集中取一条记录
		while (true) {
			if (one == null && two == null) {        //如果连个结果集都为空直接退出计算
				break;
			} else if (one == null && two != null) {    // 如果旧表中没有的记录在新表中有
				flagField = VALUE_INSERT;        // 这里说明为新数据插入了
				outputRow = two;               // 记录插入的数据内容
				two = getRowData(rsNew.resultset); // 取新表的下一条记录数据
			} else if (one != null && two == null) {    // 如果旧表中有的记录在新表中没有
				flagField = VALUE_DELETED;       // 这里说明为新数据删除了
				outputRow = one;               // 记录删除的数据内容
				one = getRowData(rsOld.resultset); // 取旧表的下一条记录数据
			} else { // 到这里才真正进入到数据比较的地方
				int compare = this.compare(one, two, keyNumbers, metaData);//比较主键自动
				if (0 == compare) { // 如果新旧两个表记录的主键值相等
                       //比较除主键外的其他字段
					int compareValues = this.compare(one, two, valNumbers, metaData);
					if (compareValues == 0) { //如果主键外的其他字段值都相同说明记录没变
						flagField = VALUE_IDENTICAL;  //记录数据记录没变标记
						outputRow = one;           //记录没变的数据内容
					} else {           //到这里说明数据更新了
						flagField = VALUE_UPDATE; //记录数据记录被更新
						outputRow = two;         //记录更新后数据内容	
					}
 
					// Get a new row from both streams...
					one = getRowData(rsOld.resultset); // 取旧表的下一条记录数据
					two = getRowData(rsNew.resultset);// 取新表的下一条记录数据
				} else {   // 如果新旧两个表记录的主键值不相等
					if (compare < 0) { // 旧表记录主键值< 新表记录主键值
						flagField = VALUE_DELETED;  //记录数据记录被删除
						outputRow = one;           //记录删除前数据内容
						one = getRowData(rsOld.resultset);// 取旧表的下一条记录数据
					} else {
						flagField = VALUE_INSERT;        //记录数据记录被添加
						outputRow = two;               //记录添加的数据内容
						two = getRowData(rsNew.resultset);// 取新表的下一条记录数据
					}
				}
 
                  //到这里，flagField 和outputRow 两个变量已经记录了一条变化量记录了。
			}
 
```

## 3、理解与总结

从原理上看，相当于将一个有序结果集RS0经insert/update/delete操作后变为另一个有序结果集RS1后，利用有序性很容易找出insert与delete的数据，利用主键相同情况下的比较再定位出update的数据。

# 五、几句尾话
上述全量算变化量的方法，本人已经编写了一个完整的DEMO程序，支持多主键的情况，下一步有时间会考虑封装为一个模块来。希望给个star哦。多谢了！

项目地址：https://gitee.com/inrgihc/mergerow  或 https://github.com/tangyibo/mergerow

原文链接：https://blog.csdn.net/inrgihc/article/details/103932231


