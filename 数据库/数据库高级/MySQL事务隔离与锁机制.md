### 概述

通常会有多个事务对同个数据表进行操作，可能会引发脏写、脏读、不可重复读、幻读这些问题。为了解决并发问题，设计了事务的隔离级别、锁机制、MVCC版本控制。

### 事务介绍

ACID

- 原子性 ：事务是一个原子操作单元,其对数据的修改,要么全都执行,要么全都不执行。
- 一致性：在事务开始和完成时,数据都必须保持一致状态。这意味着所有相关的数据规则都必须应用于事务的修改,以保持数据的完整性。
- 隔离性：数据库系统提供一定的隔离机制,保证事务在不受外部并发操作影响的“独立”环境执行。这意味着事务处理过程中的中间状态对外部是不可见的,反之亦然。
- 持久性：事务完成之后,它对于数据的修改是永久性的,即使出现系统故障也能够保持。

### 并发问题

脏读：事务A读取到了事务B未提交的数据。

不可重复读：事务A读取到了事务B已经提交的数据。

幻读：事务A读取到了事务B新增的数据，不符合隔离性。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/98786" alt="https://note.youdao.com/yws/public/resource/354ae85f3519bac0581919a458278a59/xmlnote/74624CB778F948349A31BA0A40430F51/98786" style="zoom:80%;" />

从上至下，隔离越来越强，并行化程度降低。

### 数据库锁

#### 表锁

每次操作锁住整张表。开销小，加锁快；锁定粒度大，发生锁冲突的概率最高，并发度最低；一般用在整表数据迁移的场景。

```mysql
lock table table_name read(write)
```

只有myisam引擎的表默认表锁，且只支持表锁。read锁阻塞写请求，write锁阻塞所有请求。

#### 行锁

每次操作锁住一行数据。开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度最高。

innodb支持行级锁，在repeatable_read模式下，尽可能会去选择行锁。

innodb在执行查询语句SELECT时，不会加锁，通过mvcc版本控制保证可重复读。但是update、insert、delete操作会加行锁。

#### 可重复读

事务A：

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/98800" alt="https://note.youdao.com/yws/public/resource/354ae85f3519bac0581919a458278a59/xmlnote/ED1EEDF43A8B492CAA6E140C7B10AE1F/98800" style="zoom: 67%;" />

事务B：

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/98789" alt="https://note.youdao.com/yws/public/resource/354ae85f3519bac0581919a458278a59/xmlnote/CFD200894E6A492DB64B7B8608A88649/98789" style="zoom: 67%;" />

事务A：

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/98793" alt="https://note.youdao.com/yws/public/resource/354ae85f3519bac0581919a458278a59/xmlnote/35097CC554094A4399316A1B4FFA9125/98793" style="zoom:67%;" />

发现此时事务A仍然查到的是400，是因为mvcc保证了可重复读。

此时执行

```mysql
update account set balance = balance - 50 where id = 1
```

再次查询，发现变为了300，而不是350，说明根据mvcc取得了最新的版本数据。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/98787" alt="https://note.youdao.com/yws/public/resource/354ae85f3519bac0581919a458278a59/xmlnote/7E2267C2C29E41A486D350A9DB5E24A5/98787" style="zoom:67%;" />

#### 间隙锁

假设account表里数据如下，那么间隙就有 id 为 (3,10)，(10,20)，(20,正无穷) 这三个区间。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/98874" alt="0"  />

```mysql
update account set name = 'zhuge' where id > 8 and id <18;
```

该条sql语句实际上会锁住id在(3,20)区间的所有数据，

#### 锁扩大

行级锁会通过条件找到具体某条记录，如果条件字段没有索引，可能会变成表锁。

### MVCC

MySQL在repeatable_read模式下，通过mvcc版本控制机制保证了可重复读，在read committed模式下也有mvcc(略有不同)。通过mvcc机制，避免了频繁加锁带来的开销问题，而隔离级别最高的串行化是将所有操作都加上读/写锁。

底层实现机制是通过read view和undo版本链进行比对实现的。

- repeatable_read：第一次执行select时，生成read view，此后不论数据库数据如何修改，select的结果都不会发生改，除非新开启一个事务。
- read committed：执行select时，生成一个read view，其他事务执行commit后，select又会生成新的read view。

### 结论

Innodb存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会要更高一些，但是在整体并发处理能力方面要远远优于MYISAM的表级锁定的。当系统并发量高的时候，Innodb的整体性能和MYISAM相比就会有比较明显的优势了。

- Innodb_row_lock_current_waits: 当前正在等待锁定的数量
- **Innodb_row_lock_time**: 从系统启动到现在锁定总时间长度
- **Innodb_row_lock_time_avg**: 每次等待所花平均时间
- Innodb_row_lock_time_max：从系统启动到现在等待最长的一次所花时间
- **Innodb_row_lock_waits**: 系统启动后到现在总共等待的次数

```mysql
-- 查看事务
select * from INFORMATION_SCHEMA.INNODB_TRX;
-- 查看锁
select * from INFORMATION_SCHEMA.INNODB_LOCKS;
-- 查看锁等待
select * from INFORMATION_SCHEMA.INNODB_LOCK_WAITS;

-- 释放锁，trx_mysql_thread_id可以从INNODB_TRX表里查看到
kill trx_mysql_thread_id

-- 查看锁等待详细信息
show engine innodb status\G; 
```

### 优化建议

- 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁
- 合理设计索引，尽量缩小锁的范围
- 尽可能减少检索条件范围，避免间隙锁
- 尽量控制事务大小，减少锁定资源量和时间长度，涉及事务加锁的sql尽量放在事务最后执行
- 尽可能低级别事务隔离