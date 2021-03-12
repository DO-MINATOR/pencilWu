### MySQL数据库基础思维导图

![在这里插入图片描述](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/20200629175718971.png)

### 作用

- 数据持久化
- 便于统一管理

### 概念

- DB：数据库（database）：存储数据的“仓库”。它保存了一系列有组织的数据。

- DBMS
  - 数据库管理系统（Database Management System）。数据库是通过DBMS创建和操作的容器
  - 常见的数据库管理系统：MySQL、Oracle、DB2、SqlServer等。
- SQL
  - 结构化查询语言（Structure Query Language）：专门用来与数据库通信的语言。
  - SQL的优点：①简单易学；②不是某个特定数据库供应商专有的语言，几乎所有DBMS都支持SQL；③通过灵活使用语言逻辑，可以进行非常复杂和高级的数据库操作。

### 特点

1. 库存表，表存数据
2. 表类似于java中的类，表中每列字段类似java的属性，每行数据如同一个实例

### 常见命令

```mysql
#启动客户端
mysql [-h localhost -P 3306] -uroot -proot

#查看mysql中有哪些个数据库
show databases;

#新建一个数据库
create database book;

#选择一个数据库
use test;

#查询数据表
show tables;

#查看指定的数据库中有哪些数据表
show tables from mysql;

#查询当前所在数据库
select database();

#新建一个数据表
create table math(
	id int,
	name varchar(20)
);

#查看表的结构
desc math;

#查看创建细节
show create table math;

#查看表中的所有记录
select * from math;

#向表中插入记录
insert into math (id,name) values(1,"ton");

#修改记录
update math set name="wugang" where id=1;

#删除记录
delete from math where id=1;

#删除数据表
drop table math;
truncate table math;
```

### MySQL规范

- 不区分大小写，但关键字建议大写
- 分号结尾
- 多行缩进

### SQL语言规范

- DQL：数据查询语句，主要用于检索数据
- DML：数据操纵语句，包括增删改语句
- DDL：数据库定义语句，对库、表的创建、删除、修改
- TCL：事务控制语句，包括COMMIT、ROLLBACK、SAVEPOINT等

