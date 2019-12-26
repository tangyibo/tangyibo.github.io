---
layout: post
title: GP参数修改gpconfig的使用:| greenplum
category: greenplum
tag: [greenplum]
---

这篇文章主要介绍了GP参数修改gpconfig的使用相关。
本文分为以下几个部分：
1. GPDB介绍
2. gpconfig使用


# 1.GPDB介绍
由于GPDB是分布式数据库，其参数文件分布于各个节点，如果每次参数修改都要到每个节点的配置文件上进行操作，那么效率太低，使得系统可维护性变差。所以，从4.0开始GREENPLUM就提供工具gpconfig，通过主节点就可以对所有节点的参数文件进行批量修改，极大方便了DBA的工作。

# 2.gpconfig使用

## 2.1 修改参数
比如要修改最大连接数的参数，可以以gpadmin用户登录master，执行如下命令:

```
gpconfig -c max_connection -v 750 -m 150
```

说明如下：

- (1) -c 指定要参数;
- (2) -v 指定segment 参数的设置值，如果没有-m参数，它也指定master上参数的设置，-m 如果希望master参数不同于segment，那么通过该参数独立指定master的参数值。

## 2.2 查询参数

可以使用指令gpconfig -s 查询参数，例如查看节点的最大连接数：

```
gpconfig -s max_connections 
```

## 2.3 注意事项

gpconfig只能在系统启动的情况下调用，所以如果参数修改不合适，导致系统无法启动时，我们可以用下列方法处理:
- 1、先把master的参数修改成正常的值
- 2、gpstart -m 仅启动master进入管理模式
- 3、gpconfig -r  <参数>   -- 把参数重置成默认值
- 4、gpstop -a -r -M fast
