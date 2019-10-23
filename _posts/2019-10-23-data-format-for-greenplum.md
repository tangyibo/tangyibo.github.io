---
layout: post
title:greenplum的存储格式|greenplum数据库 
category: Greenplum
tag: [Greenplum]
---

这篇文章主要记录了Greenplum数据库的数据存储格式相关。
本文分为以下几个部分：
1. 存储格式介绍
2. Heap表
3. AO表


## Greenplum数据库的数据存储格式

### 存储格式介绍

Greenplum（以下简称GP）有2种存储格式，Heap表和AO表（AORO表，AOCO表）。

- Heap表：这种存储格式是从PostgreSQL继承而来的，目前是GP默认的表存储格式，只支持行存储。
- AO表：   AO表最初设计是只支持append的（就是只能insert），因此全称是Append-Only，在4.3之后进行了优化，目前已经可以update和delete了，全称也改为Append-Optimized。AO支持行存储（AORO）和列存储（AOCO）。

### Heap表
Heap表是从PostgreSQL继承而来，使用MVCC来实现一致性。如果你在创建表的时候没有指定任何存储格式，那么GP就会使用Heap表。

Heap表支持分区表，只支持行存，不支持列存和压缩。需要注意的是在处理update和delete的时候，Heap表并没有真正删除数据，而只是依靠version信息屏蔽老的数据，因此如果你的表有大量的update或者delete，表占用的物理空间会不断增大，这个时候需要依靠vacuum来清理老数据。

Heap表不支持逻辑增量备份，因此如果要对Heap表做快照，每次都需要导出全量数据。

建表语句
```
CREATE TABLE heap(
	a int,
	b varchar(32)
) DISTRIBUTED BY (a);
```
- 最佳实践：

>如果该表是一张小表，比如数仓中的维度表，或者数据量在百万以下，推荐使用Heap表。
>如果该表的使用场景是OLTP的，比如有较多的update和delete，查询多是带索引的点查询等，推荐使用Heap表。


### AO表
AO表是GP特有的，设计的目的就是为了数仓中大型的事实表。AO表支持行存和列存，并且也支持对数据进行压缩。

AO表无论是在表的逻辑结构还是物理结构上，都与Heap表有很大的不同。比如上文所述Heap表使用MVCC控制update和delete之后数据的可见性，而AO表则使用一个附加的bitmap表来实现，这个表的的内容就是表示AO表中哪些数据是可见的。

对于有大量update和delete的AO表，同样需要vacuum进行维护，不过在AO表中，vacuum需要对bitmap进行重置并压缩物理文件，因此通常比Heap的vacuum要慢。

### 参考地址
https://cloud.tencent.com/developer/article/1393372?from=10680