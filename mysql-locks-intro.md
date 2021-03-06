---
layout: post
comments: true
title: mysql锁介绍（一）
date: 2017-02-24 09:36:41
tags:
- mysql
categories:
- mysql
---

> [数据库的读现象浅析](http://www.hollischuang.com/archives/900)中介绍过，在并发访问情况下，可能会出现脏读、不可重复读和幻读等读现象，为了应对这些问题，主流数据库都提供了锁机制，并引入了事务隔离级别的概念。

### 并发控制

> 在计算机科学，特别是程序设计、操作系统、多处理机和数据库等领域，并发控制（Concurrency control）是确保及时纠正由并发操作导致的错误的一种机制。

数据库管理系统（DBMS）中的并发控制的任务是确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性和统一性以及数据库的统一性。下面举例说明并发操作带来的数据不一致性问题：

现有两处火车票售票点，同时读取某一趟列车车票数据库中车票余额为 X。两处售票点同时卖出一张车票，同时修改余额为 X -1写回数据库，这样就造成了实际卖出两张火车票而数据库中的记录却只少了一张。 产生这种情况的原因是因为两个事务读入同一数据并同时修改，其中一个事务提交的结果破坏了另一个事务提交的结果，导致其数据的修改被丢失，破坏了事务的隔离性。并发控制要解决的就是这类问题。

封锁、时间戳、乐观并发控制(乐观锁)和悲观并发控制（悲观锁）是并发控制主要采用的技术手段。


### 锁

当并发事务同时访问一个资源时，有可能导致数据不一致，因此需要一种机制来将数据访问顺序化，以保证数据库数据的一致性。锁就是其中的一种机制。

在计算机科学中，锁是在执行多线程时用于强行限制资源访问的同步机制，即用于在并发控制中保证对互斥要求的满足。

### 锁的分类(oracle)

一、按操作划分，可分为DML锁、DDL锁

二、按锁的粒度划分，可分为表级锁、行级锁、页级锁（mysql）

三、按锁级别划分，可分为共享锁、排他锁

四、按加锁方式划分，可分为自动锁、显示锁

五、按使用方式划分，可分为乐观锁、悲观锁


DML锁（data locks，数据锁），用于保护数据的完整性，其中包括行级锁(Row Locks (TX锁))、表级锁(table lock(TM锁))。 DDL锁（dictionary locks，数据字典锁），用于保护数据库对象的结构，如表、索引等的结构定义。其中包排他DDL锁（Exclusive DDL lock）、共享DDL锁（Share DDL lock）、可中断解析锁（Breakable parse locks）





