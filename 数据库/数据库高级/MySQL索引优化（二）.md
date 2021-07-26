### L示例表

```mysql
CREATE TABLE `employees` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(24) NOT NULL DEFAULT '' COMMENT '姓名',
  `age` int(11) NOT NULL DEFAULT '0' COMMENT '年龄',
  `position` varchar(20) NOT NULL DEFAULT '' COMMENT '职位',
  `hire_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间',
  PRIMARY KEY (`id`),
  KEY `idx_name_age_position` (`name`,`age`,`position`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='员工记录表';
```

### Limit

```mysql
select * from employees limit 10000,10;
```

该SQL语句由于无法走索引。因此是直接从内存中先load10010条数据，然后取后10条，因此type为all。

**1、通过索引+范围优化查询**

```mysql
select * from employees where id > 90000 limit 5;
```

如果，id为主键，则可以通过where条件查找出该范围内的数据，然后去除前5条。

但是这种优化方式条件苛刻，需要前面的数据完整且索引连续。



**2、覆盖索引引导limit走索引查询**

```mysql
select * from employees ORDER BY name limit 90000,5;
```

![image-20210720113752751](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210720113752751.png)

该查询没有走索引，而是all+filesort，可以通过覆盖索引优化，其实优化效率还是极高的：

```mysql
select age,name,positi	on from employees ORDER BY name limit 90000,5;
```

![image-20210720113814755](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210720113814755.png)

如何不走覆盖索引，且又让它利用上联合索引呢？方法是，先让二级索引查找出id，再用关联查询满足id值的记录。

```mysql
select * from tt inner join (select id from tt where age>10 limit 9000,5) d where tt.id=d.id;
```

![image-20210720120246970](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210720120246970.png)

1. derived表先根据索引找出5条记录，查询类型是range。
2. primary先将临时表\<derived2>遍历查询all。
3. tt表再根据id进行主键的聚集索引查询。

### join

**示例表**

```mysql
CREATE TABLE `t1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_a` (`a`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

- t1大表：10000条数据
- t2小表：100条数据

#### NLJ嵌套循环连接

mysql优化器将取出来的字段较少的表作为**驱动表**，去查找另一张表。可通过straight jion指定驱动表（左边的为驱动表）。每次从驱动表中取出一行数据，根据关联字段再另一张表中取出满足条件的记录，再做合并。

```mysql
select * from t1 inner join t2 on t1.a= t2.a;
```

![https://note.youdao.com/yws/public/resource/df15aba3aa76c225090d04d0dc776dd9/xmlnote/086C5EE10BB44DB5BC6AF3D21F97B75A/100112](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/100112)

- 可以看到，t2作为了驱动表，该表使用all查询，之后再在t1中做条件查询，走二级索引。
- inner join自动选择驱动表，straight join指定驱动表，left join左边为驱动表，right join右边为驱动表。
- 整个过程中，t2扫描了100次，每次又从t1走二级索引，ref是常数级查找，因此总共时间复杂度约为100。

#### BNL块嵌套循环连接

```mysql
select * from t1 inner join t2 on t1.b= t2.b;
```

![https://note.youdao.com/yws/public/resource/df15aba3aa76c225090d04d0dc776dd9/xmlnote/AA61DB497D1D4438BFB5C17EEB3077C1/100111](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/100111)

- t2所有数据导入到join buffer中。
- t1数据导入内存，与jion buffer做比对。
- 整个过程，一共扫描了10000+100行记录，而判断字段是否相等次数为10000*100。

如果此时采用NLJ算法，则扫描次数都将变为10000*100次。IO时间大大加长。

#### 总结

- 尽量给关联字段添加索引，即NLJ算法。
- 小表驱动大表，一般MySQL优化器会自动分析，万不得已才使用straight join。
- 同第二条，将小表放入join_buffer中，使得扫描次数减少。

### in和exists

原则依然是小表驱动大表。

in后面的查询先执行。

```mysql
select * from A where id in (select id from B)  
#等价于：
　　for(select id from B){
      select * from A where A.id = B.id
    }
```

exists前面的查询先执行。

```mysql
select * from A where exists (select 1 from B where B.id = A.id)
#等价于:
    for(select * from A){
      select * from B where B.id = A.id
    }
#EXISTS (subquery)只返回TRUE或FALSE,因此子查询中的SELECT * 也可以用SELECT 1替换,官方说法是实际执行时会忽略SELECT清单,因此没有区别
```

### count

```mysql
mysql> EXPLAIN select count(1) from employees;
mysql> EXPLAIN select count(id) from employees;
mysql> EXPLAIN select count(name) from employees;
mysql> EXPLAIN select count(*) from employees;
```

count(字段)排除字段为null的记录。

有二级索引：`count(*)`和`count(1)`最快，`count(id)`和`count(字段)`较慢，因为覆盖索引数据量较小，而且不用比对某一字段是否为null，`count(id)`走聚集索引，直接累加计数，`count(字段)`走覆盖索引，同时比对字段。

无二级索引：`count(*)`和`count(1)`依然最快，虽然也要走聚集索引，但不需要像`count(id)`比对id字段，`count(字段)`最慢，要走all扫描的同时，还需要比对字段。

#### 一些优化手段

myisam引擎会自动维护总记录数，所以结果为常量读取。

```mysql
select count(*) from test;
#test为myisam引擎存储
```

![https://note.youdao.com/yws/public/resource/df15aba3aa76c225090d04d0dc776dd9/xmlnote/8CFEBA7E11E64D30938E45D3D2D5AFD3/100114](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/100114)

innodb由于有**mvcc**机制，使得count需要实时计算。

估值计算可以采取如下sql查询。

```mysql
show table status like '表名';
```

维护总数到redis中，在内存中使用原子自增，效率很高，但存在一致性问题。

通过事务，没增删一条数据，就在记录数上+1/-1，效率次之。

### 阿里巴巴MySQL规约解读

#### 数据类型选择

1. 确定合适的类型，数字、字符串、时间。
2. 数字确定有无符号，字符串确定定长、边长、字节数。
3. 尽量增加not null约束。

**日期和时间**

|   类型    | 字节 | 范围                                       | 格式                | 用途                     |
| :-------: | :--: | ------------------------------------------ | ------------------- | ------------------------ |
|   DATE    |  3   | 1000-01-01 到 9999-12-31                   | YYYY-MM-DD          | 日期值                   |
|   TIME    |  3   | '-838:59:59' 到 '838:59:59'                | HH:MM:SS            | 时间值或持续时间         |
|   YEAR    |  1   | 1901 到 2155                               | YYYY                | 年份值                   |
| DATETIME  |  8   | 1000-01-01 00:00:00 到 9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值         |
| TIMESTAMP |  4   | 1970-01-01 00:00:00 到 2038-01-19 03:14:07 | YYYYMMDDhhmmss      | 混合日期和时间值，时间戳 |

1. MySQL能存储的最小时间粒度为秒。
2. 建议用DATE数据类型来保存日期。MySQL中默认的日期格式是yyyy-mm-dd。
3. 用MySQL的内建类型DATE、TIME、DATETIME来存储时间，而不是使用字符串。
4. 当数据格式为TIMESTAMP和DATETIME时，可以用CURRENT_TIMESTAMP作为默认（MySQL5.6以后），MySQL会自动返回记录插入的确切时间。
5. TIMESTAMP是UTC时间戳，与时区相关，而DATETIME不同时区读出来的时间不一样，业务上需要做转换。
6. DATETIME的存储格式是一个YYYYMMDD HH:MM:SS的整数，与时区无关，你存了什么，读出来就是什么。

**字符串**

|    类型    | 大小                | 用途                                                         |
| :--------: | :------------------ | :----------------------------------------------------------- |
|    CHAR    | 0-255字节           | 定长字符串，char(n)当插入的字符数不足n时(n代表字符数)，插入空格进行补充保存。在进行检索时，尾部的空格会被去掉。 |
|  VARCHAR   | 0-65535 字节        | 变长字符串，varchar(n)中的n代表最大字符数，插入的字符数不足n时不会补充空格 |
|  TINYBLOB  | 0-255字节           | 不超过 255 个字符的二进制字符串                              |
|  TINYTEXT  | 0-255字节           | 短文本字符串                                                 |
|    BLOB    | 0-65 535字节        | 二进制形式的长文本数据                                       |
|    TEXT    | 0-65 535字节        | 长文本数据                                                   |
| MEDIUMBLOB | 0-16 777 215字节    | 二进制形式的中等长度文本数据                                 |
| MEDIUMTEXT | 0-16 777 215字节    | 中等长度文本数据                                             |
|  LONGBLOB  | 0-4 294 967 295字节 | 二进制形式的极大文本数据                                     |
|  LONGTEXT  | 0-4 294 967 295字节 | 极大文本数据                                                 |

1. 字符串的长度相差较大用VARCHAR；字符串短，且所有值都接近一个长度用CHAR。
2. CHAR适用于包括人名、邮政编码、电话号码和不超过255个字符长度的任意字母数字组合。而VARCHAR对于长字符串且空间敏感支持较好，但是每次读取和写入时会做额外转换，时间换空间。
3. 尽量少用BLOB和TEXT，如果实在要用可以考虑将BLOB和TEXT字段单独存一张表，用id关联。
4. BLOB系列存储二进制字符串，与字符集无关。TEXT系列存储字符串，与字符集相关。
5. utf8三个字节代表一个字符，uft8mb4，四个字节代表一字符（支持Emoji）

