### Explain介绍

在常规SQL语句之前添加Explain关键字，可以检查该SQL语句的执行特性，帮助分析性能瓶颈。

![image-20210718175150402](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718175150402.png)

### 表示例

```mysql
CREATE TABLE `actor` (
`id` int(11) NOT NULL,
`name` varchar(45) DEFAULT NULL,
`update_time` datetime DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

actor表，唯一主键id

```mysql
CREATE TABLE `film` (
 `id` int(11) NOT NULL AUTO_INCREMENT,
 `name` varchar(10) DEFAULT NULL,
 PRIMARY KEY (`id`),
 KEY `idx_name` (`name`)
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

film表，主键索引id，二级索引name，由于二级索引叶节点包含主键值，因此可以看作是聚集索引，查询时无需回表。

```mysql
CREATE TABLE `film_actor` (
 `id` int(11) NOT NULL,
 `film_id` int(11) NOT NULL,
 `actor_id` int(11) NOT NULL,
 `remark` varchar(255) DEFAULT NULL,
 PRIMARY KEY (`id`),
 KEY `idx_film_actor_id` (`film_id`,`actor_id`)
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

film_actor表，主键索引id，联合索引film_id，actor_id。

### 列说明

#### id

一般有几个select，就有几个id，值越大，优先级越高，为null的最后执行。

#### select_type

- simple：简单查询，不包括子查询和union。
- primary：复杂查询中的最外层查询。
- subquery：包含在select中的子查询（不在from子句中）
- derived：在from子句中的子查询，这类查询会生成一张临时表，因此有别于subquery。

```mysql
explain select (select 1 from actor where id = 1) from (select * from film where id = 1) der;
```

![image-20210718180334468](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718180334468.png)

#### table

当前sql语句查询的表。

#### type

决定如何从数据表中查找数据，性能从高到低依次为：

`null > system > const > eq_ref > ref > range > index > ALL`

最佳实践是，将范围优化到range以上。

- null：这类查找一般直接通过索引就可以获得结果。

```mysql
explain select min(id) from film;
```

![image-20210718201849136](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718201849136.png)

- const，system：主键索引或唯一索引查找时，直接取得一条数据，这样的查找也十分快速。

```mysql
explain extended select * from (select * from film where id = 1) tmp;
```

![image-20210718202125451](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718202125451.png)

- eq_ref：也是走主键索引或唯一索引，相比const，system，会多次走const，这样的“批量的const，system"查询就会转变为eq_ref。

```mysql
select * from film_actor left join film on film_actor.film_id = film.id
```

![image-20210718202944134](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718202944134.png)

- ref：不使用唯一索引，而是使用二级索引或索引前缀(列)，这样单次查询就会返回多个记录。

```mysql
explain select * from film where name = 'film1';
```

![image-20210718203011342](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718203011342.png)

```mysql
explain select * from film left join film_actor on film.id = film_actor.film_id;
```

![image-20210718203323731](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718203323731.png)

由于通过film_id查询film_actor表时，走的联合索引的第一列，因此一次查询会返回多条记录，所以也是ref。

- range：走索引，且是对索引的一个范围查找。

```mysql
 explain select * from actor where id > 1
```

![image-20210718203557296](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718203557296.png)

- index：扫描全表索引，通常为二级索引，只要所需的结果能在二级索引中找到，而不需要回表，就是**覆盖索引**，相比all，也是全表扫描，但聚集索引有可能比二级索引大，关键看叶节点字段大小。

```mysql
explain select * from film;
```

![image-20210718203931866](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718203931866.png)

这里film的name为二级索引，通过对二级索引叶节点全扫描，可以得到name和id字段，满足要求，即覆盖索引，因此为index。

- all：全表扫描，IO开销最高的查询种类，需优化。

```mysql
explain select * from actor;
```

![image-20210718204136870](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718204136870.png)

#### possible_keys

优化阶段时分析得到可能会用到的索引，但最终是否使用该列中所列到的索引，还需要结合cost综合判断。

#### key

查询阶段所真正采取的索引列。

#### key_len

在具体使用的索引列中，又具体采取了哪些字段，甚至是几个字符，一般为联合索引或字符串索引，符合左前列、左前缀原则。

```mysql
explain select * from film_actor where film_id = 2;
```

![image-20210718204803853](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718204803853.png)

索引最大长度是768B，当字符串过长时，mysql会做一个类似左前缀索引的处理，将前半部分的字符提取出来做索引。

#### ref

在根据索引查找时，=右边对应的是常量（const）还是字段（列入film.id）

#### rows

估计要读取并检测的行数

#### Extra

额外信息

- Using index：使用了索引

```mysql
explain select film_id from film_actor where film_id = 1;
```

![image-20210718205558544](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718205558544.png)

- Using where：使用了where条件，且没有走索引。

```mysql
explain select * from actor where name = 'a';
```

![image-20210718205911848](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718205911848.png)

- Using index condition：使用了索引，但通常是前列（缀）索引

```mysql
explain select * from film_actor where film_id > 1;
```

![image-20210718210118708](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718210118708.png)

- Using temporary：使用了临时表

```mysql
explain select distinct name from actor;
```

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718210337475.png" alt="image-20210718210337475" style="zoom:110%;" />

可以增加索引优化，例如给name加上二级索引

![image-20210718210435262](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718210435262.png)

- Using filesort：直接对原始数据进行排序而不利用索引排序，数据较小时导入内存进行排序，数据较大是外部排序，极耗性能。

```mysql
explain select * from actor order by name;
```

![image-20210718210655826](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718210655826.png)

同样，给name列增加二级索引，优化后为

![image-20210718210757970](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718210757970.png)

- select tables optimized away：访问索引，作聚合函数，如max、min

```mysql
 explain select min(id) from film;
```

![image-20210718211053475](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718211053475.png)

### 最佳实践

#### 表示例

```mysql
CREATE TABLE `employees` (
 `id` int(11) NOT NULL AUTO_INCREMENT,
 `name` varchar(24) NOT NULL DEFAULT '' COMMENT '姓名',
 `age` int(11) NOT NULL DEFAULT '0' COMMENT '年龄',
 `position` varchar(20) NOT NULL DEFAULT '' COMMENT '职位',
 `hire_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间',
 PRIMARY KEY (`id`),
 KEY `idx_name_age_position` (`name`,`age`,`position`) USING BTREE
 ) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 COMMENT='员工记录表';
```

#### 左前列原则

尽量按照联合索引的先后顺序组织索引，如果用name、position作查询条件，则只会用name做索引。

```mysql
SELECT * FROM employees WHERE name= 'LiLei';
```

![image-20210718211240318](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718211240318.png)

```mysql
SELECT * FROM employees WHERE name= 'LiLei' AND age = 22;
```

![image-20210718211254405](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718211254405.png)

```mysql
SELECT * FROM employees WHERE name= 'LiLei' AND age = 22 AND position ='manager';
```

![image-20210718211314499](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718211314499.png)

#### 查询列做函数转换失去索引性

否则，将导致失去索引效果，转而全表扫描

```mysql
SELECT * FROM employees WHERE left(name,3) = 'LiLei';
```

![image-20210718211651438](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718211651438.png)

```mysql
 ALTER TABLE `employees` ADD INDEX `idx_hire_time` (`hire_time`) USING BTREE ;
 EXPLAIN select * from employees where date(hire_time) ='2018‐09‐30';
```

![image-20210718211806746](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718211806746.png)

给日期增加了二级索引，但仍是全表扫描，因为函数作用导致索引失去有序性。

```mysql
EXPLAIN select * from employees where hire_time >='2018‐09‐30 00:00:00' and hire_time < ='2018‐09‐30 23:59:59';
```

改善查询方式，这种情况下，就有可能为range查询，同时using index。

#### 范围查询列右边的列失去索引性

```mysql
SELECT * FROM employees WHERE name= 'LiLei' AND age > 22 AND position ='manager';
```

![image-20210718212223910](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718212223910.png)

#### 尽量使用覆盖索引

即减少回表次数，避免出现`select *` 的情况。

```mysql
SELECT * FROM employees WHERE name= 'LiLei' AND age = 23 AND position ='manager';
```

![image-20210718212451646](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718212451646.png)

```mysql
SELECT name,age FROM employees WHERE name= 'LiLei' AND age = 23 AND position ='manager';
```

![image-20210718212508269](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718212508269.png)

#### 查询列如果是!=、not in、in时失去索引性

同理，这些查询条件无法利用到索引的有序性，将导致all扫描，另外，即使是<、>、<=、>=这些，最终也会根据cost值来决定是否采用索引，譬如查询范围很大时，不如直接全表扫描更直接。

```mysql
SELECT * FROM employees WHERE name != 'LiLei';
```

![image-20210718212747301](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718212747301.png)

#### like并以通配符开头的查询失去索引性

```mysql
SELECT * FROM employees WHERE name like '%Lei'
```

![image-20210718212854557](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718212854557.png)

```mysql
SELECT * FROM employees WHERE name like 'Lei%'
```

![image-20210718212921722](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718212921722.png)

```mysql
SELECT * FROM employees WHERE name like '%Lei%';
#通常这条语句只能这么优化
SELECT name,age,position FROM employees WHERE name like '%Lei%'
#从all到index，走覆盖索引
```

![image-20210718213213105](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718213213105.png)

#### 字符串不加引号失去索引性

由于MySQL内部做了一次数据类型转换，将导致失去有序性。

```mysql
SELECT * FROM employees WHERE name = 1000;
```

![image-20210718213426240](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210718213426240.png)

### 总结

假设联合索引列为`index(a,b,c)`

| where                         | 索引列 |
| ----------------------------- | ------ |
| a=3                           | a      |
| a=3 and b=5                   | a b    |
| a=3 and b=5 and c=4           | a b c  |
| b=3                           | /      |
| b=3 and c=4                   | /      |
| c=4                           | /      |
| a=3 and c=5                   | a      |
| a=3 and b>5 and c=5           | a b    |
| a=3 and c=3 and b=5           | a b c  |
| a=3 and b like 'kk%' and c=4  | a b    |
| a=3 and b like '%kk' and c=4  | a      |
| a=3 and b like 'k%kk' and c=4 | a b    |