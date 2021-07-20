### 示例表

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

