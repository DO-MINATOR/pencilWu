### 数值型

#### 整型

tinyint、smallint、mediumint、int/integer、bigint
1	       2		   3	       4		 8

1. 如不设置无符号还是有符号，则默认有符号，可手动设置unsigned
2. 如果插入数值超出范围，则插入临界值
3. 长度仅和显示有关，存储范围只与类型有关，长度是关于零填充，需要配合zerofill使用

```mysql
#1.如何设置无符号和有符号
CREATE TABLE tab_int (t1 INT (7) ZEROFILL, t2 INT (7) ZEROFILL) ;
```

#### 浮点型

定点数：dec(M,D)；decimal(M,D)
浮点数：float(M,D) 4；double(M,D) 8

1. M代表所有的位数，D代表小数部分位数
2. 定点数需要指明M，D
3. 当精度要求较高时，选择定点数

```mysql
CREATE TABLE tab_float (f1 FLOAT, f2 DOUBLE, f3 DECIMAL) ;
```

### 字符型

短文本：char、varchar

| 写法       | M          | 特点     | 空间耗费 | 效率 |
| ---------- | ---------- | -------- | -------- | ---- |
| char(M)    | 最大字符数 | 固定长度 | 高       | 高   |
| varchar(M) | 最大字符数 | 可变长度 | 低       | 低   |

长文本：text、blob(二进制)

### 日期型

- date（日期）
- time（仅时间）
- year
- datetime日期+时间，不随时区改变而改变
- timestamp日期+时间，随时区改变而改变

### TCL（事务）

事务控制语句，事务指一个或一组SQL语句组合为一个单元，这个单元要么全部执行，要么全都不执行。针对DQL、DML语句生效，DDL无法通过事务控制

```mysql
/*案例：转账
张三丰		1000
郭襄		1000*/
update 表 set 张三丰的余额=500 where name='张三丰'
意外
update 表 set 郭襄的余额=1500 where name='郭襄'
```

ACID：

- 原子性：一个事务不可再分割，要么都执行要么都不执行（事务的提交与回滚）
- 一致性：一个事务执行会使数据从一个一致状态切换到另外一个一致状态（银行总金额一致）
- 隔离性：一个事务的执行不受其他事务的干扰（MySQL四大隔离级别）
- 持久性：一个事务一旦提交，则会永久的改变数据库的数据

```mysql
#原子性测试
步骤1：关闭事务
set autocommit=0;
start transaction;可选的

步骤2：编写事务中的sql语句(select insert update delete)
语句1;
语句2;
...
步骤3;
commit;提交事务
rollback;回滚事务
```

**并发事务：**

事务的并发问题是指当多个事务同时操作同一个数据库时导致的。常见的并发问题有：

- 脏读：一个事务如果读取到其他还没有提交的事务，如果后续rollback，那么此次读就是脏读
- 不可重复读：一个事务读取的结果不一样（行数据的修改）
- 幻读：一个事务读取的结果不一样（行数据的增删）

事务的隔离性就是保证数据库的事务避免产生上述问题，MySQL有四种隔离级别(默认repeatable read)，隔离性越强，并发性越弱。

|                  | 脏读 | 不可重复读 | 幻读 |
| ---------------- | ---- | ---------- | ---- |
| read uncommitted | √    | √          | √    |
| read committed   | ×    | √          | √    |
| repeatable read  | ×    | ×          | √    |
| serializable     | ×    | ×          | ×    |

```mysql
查看隔离级别
select @@tx_isolation;
设置隔离级别
set session|global transaction isolation level 隔离级别;
```

**注意：**所谓的隔离是在不同事务下的操作，如当两个事务都处于开启状态(SET autocommit=0且START TRANSACTION)，此时两个事务处于repeatable read模式时，事务A对数据的修改不会对事务B产生影响，但A对数据的增删则会对事务B产生影响。

**保存点：**

也是在事务下设置的，如

```mysql
SET autocommit=0;
START TRANSACTION;

DELETE FROM account WHERE id=25;
SAVEPOINT a;#设置保存点
DELETE FROM account WHERE id=28;
ROLLBACK TO a;#回滚到保存点

SELECT * FROM account;
```

### 视图

一种虚拟表，是通过sql查询语句动态生成的，保存了sql执行的逻辑结果，并不保存真实数据，适用于频繁查询一个结果集，且该结果的查询本身就很复杂。

`CREATE VIEW view AS 查询语句...`

```mysql
CREATE VIEW v1 AS
SELECT stuname,majorname
FROM stuinfo s
INNER JOIN major m ON s.`majorid`= m.`id`;

SELECT * FROM v1 WHERE stuname LIKE '张%';
```

- 重用了sql执行语句
- 简化复杂操作，逻辑更见鲜明
- 保护数据，提高安全性

重新定义该视图的语句是

`ALTER VIEW view AS 查询语句;`

删除视图（多个）

`DROP VIEW view...`

**注意：**视图一般用于查询，不做更新，更新有可能成功，且会将更新结果写回到原始数据表中。

和表的对比：

|      | 关键字 | 占用物理空间 | 目的 |
| ---- | ------ | ------------ | ---- |
| 表   | table  | 真实数据     | CRUD |
| 视图 | view   | sql逻辑      | 查询 |

