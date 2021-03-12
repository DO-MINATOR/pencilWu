### DML

数据操纵语句

- insert：插入
- update：更新
- delete：删除

#### insert

`insert into 表名[(字段名,…)] values(值,…);`字段个数、参数需一致，如果不写字段名，则默认所有列

```mysql
SELECT * FROM beauty;
#1.插入的值的类型要与列的类型一致或兼容
INSERT INTO beauty(id,NAME,sex,borndate,phone,photo,boyfriend_id)
VALUES(13,'唐艺昕','女','1990-4-23','1898888888',NULL,2);

#2.不可以为null的列必须插入值。可以为null的列如何插入值？
    #方法一：写null
    INSERT INTO beauty(id,NAME,sex,borndate,phone,photo,boyfriend_id)
    VALUES(13,'唐艺昕','女','1990-4-23','1898888888',NULL,2);

    #方法二：都不写
    INSERT INTO beauty(id,NAME,sex,phone)
    VALUES(15,'娜扎','女','1388888888');

#3.列的顺序可以调换
INSERT INTO beauty(NAME,sex,id,phone)
VALUES('蒋欣','女',16,'110');

#4.列数和值的个数必须一致
INSERT INTO beauty(NAME,sex,id,phone)
VALUES('关晓彤','女',17,'110');

#5.可以省略列名，默认所有列，而且列的顺序和表中列的顺序一致
INSERT INTO beauty
VALUES(18,'张飞','男',NULL,'119',NULL,NULL);

#6.同时添加多行数据
INSERT INTO beauty
VALUES(23,'唐艺昕1','女','1990-4-23','1898888888',NULL,2),
(24,'唐艺昕2','女','1990-4-23','1898888888',NULL,2),
(25,'唐艺昕3','女','1990-4-23','1898888888',NULL,2);

#7.支持子查询
INSERT INTO beauty(id,NAME,phone) SELECT 26,'宋茜','11809866';

INSERT INTO beauty(id,NAME,phone) 
SELECT id,boyname,'1234567' FROM boys WHERE id<3;
```

#### update

`update 表名 set 列=新值,列=新值,… where 筛选条件;`

```mysql
#1.修改单表的记录
#修改beauty表中姓唐的女神的电话为13899888899
UPDATE beauty SET phone = '13899888899'
WHERE NAME LIKE '唐%';

#2.（同时）修改多表的记录
#案例 1：修改张无忌的女朋友的手机号为114
UPDATE boys bo
INNER JOIN beauty b ON bo.`id`=b.`boyfriend_id`
SET b.`phone`='119',bo.`userCP`=1000
WHERE bo.`boyName`='张无忌';

#案例2：修改没有男朋友的女神的男朋友编号都为2号
UPDATE boys bo
RIGHT JOIN beauty b ON bo.`id`=b.`boyfriend_id`
SET b.`boyfriend_id`=2 WHERE bo.`id` IS NULL;
```

#### delete

单表删除：`delete from 表名 where 筛选条件`
多表删除：`delete 表1的别名,表2的别名 from 别名1 inner|left|right join 别名2 on 连接条件 where 筛选条件;`

```mysql
#1.单表的删除
#案例：删除手机号以9结尾的女神信息
DELETE FROM beauty WHERE phone LIKE '%9';
SELECT * FROM beauty;

#2.多表的删除
#案例：删除张无忌的女朋友的信息
DELETE b
FROM beauty b
INNER JOIN boys bo ON b.`boyfriend_id` = bo.`id`
WHERE bo.`boyName`='张无忌';

#案例：删除黄晓明的信息以及他女朋友的信息
DELETE b,bo
FROM beauty b
INNER JOIN boys bo ON b.`boyfriend_id`=bo.`id`
WHERE bo.`boyName`='黄晓明';
```

1. delete 可以加where 条件，truncate全部删除。
2. 假如要删除的表中有自增长列，如果用delete删除后，再插入数据，自增长列的值从断点开始，而truncate删除后，再插入数据，自增长列的值从1开始。
3. truncate删除没有返回值，delete删除有返回值（删除的行数）。
4. truncate删除不能回滚，delete删除可以回滚。

### DDL

**库管理**

#### 创建库

```mysql
#案例：创建库Books
CREATE DATABASE IF NOT EXISTS books 【character set 字符集名】;
```

#### 修改库

```mysql
#案例：更改库的字符集
ALTER DATABASE books CHARACTER SET gbk;
```

#### 删除库

```mysql
#案例：库的删除
DROP DATABASE IF EXISTS books;
```

**表管理**

#### 表创建

```mysql
/*
语法：
create table 表名(
	列名 列的类型【(长度) 约束】,
	列名 列的类型【(长度) 约束】,
	列名 列的类型【(长度) 约束】,
	...
	列名 列的类型【(长度) 约束】
)
*/
#案例：创建表Book
CREATE TABLE book (
  id INT,
  bName VARCHAR (20),
  price DOUBLE,
  authorId INT,
  publishDate DATETIME
);
```

#### 表修改

```mysql
1.添加列
alter table 表名 add column 列名 类型 【first|after 字段名】;
2.修改列的类型或约束
alter table 表名 modify column 列名 新类型 【新约束】;
3.修改列名
alter table 表名 change column 旧列名 新列名 类型;
4 删除列
alter table 表名 drop column 列名;
5.修改表名
alter table 表名 rename 【to】 新表名;
```

#### 表删除

```mysql
DROP TABLE IF EXISTS book_author;
```

#### 表复制

```mysql
1、复制表的结构
create table 表名 like 旧表;
2、复制表的结构+数据
create table 表名 
select 查询列表 from 旧表【where 筛选】;

#案例
#1.仅仅复制表的结构
CREATE TABLE copy LIKE author;

#2.复制表的结构+数据
CREATE TABLE copy2 SELECT * FROM author;

#只复制部分数据
CREATE TABLE copy3 SELECT id,au_name
FROM author WHERE nation='中国';

#仅仅复制某些字段且不要数据
CREATE TABLE copy4 SELECT id,au_name
FROM author WHERE 0;
```

### 约束

也属于DDL语言，类似于字段类型，也是用于限制表中的数据。

#### **六大约束：**

- NOT NULL：非空
- DEFAULT：默认值
- PRIMARY KEY：主键，默认非空、唯一
- UNIQUE：唯一，只能有一行数据存在空值
- CHECH：检查约束（MySQL中无效，推荐使用enum、set代替）
- FOREIGN KEY：外键，从表的外键值只能来子主表的关联列（一般为主键、唯一键）

约束的创建时机为**创建表**和**修改表**
约束的创建位置为**列级约束**和**表级约束**

1、创建表时创建约束的语法

```mysql
CREATE TABLE 表名{
	字段名 字段类型 列级约束,
	字段名 字段类型,
	表级约束
};
```

```mysql
列级约束
CREATE TABLE stuinfo(
	id INT PRIMARY KEY,#主键
	stuName VARCHAR(20) NOT NULL UNIQUE,#非空
	gender enum('男','女'),#代替check
	seat INT UNIQUE,#唯一
	age INT DEFAULT 18,#默认约束
	majorId INT REFERENCES major(id)#外键
);

CREATE TABLE major(
	id INT PRIMARY KEY,
	majorName VARCHAR(20)
);

#查看stuinfo中的所有键，包括主键、外键、唯一
SHOW INDEX FROM stuinfo;

表级约束
CREATE TABLE stuinfo(
	id INT,
	stuname VARCHAR(20),
	gender CHAR(1),
	seat INT,
	age INT,
	majorid INT,
	
	CONSTRAINT pk PRIMARY KEY(id),#主键
	CONSTRAINT uq UNIQUE(seat),#唯一键
	CONSTRAINT ck CHECK(gender ='男' OR gender  = '女'),#检查
	CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id)#外键	
);
```

2、修改表时修改约束的语法

```mysql
1、修改列级约束
alter table 表名 modify column 字段名 字段类型 新约束;

2、添加表级约束
alter table 表名 add  PRIMARY KEY/INDEX/FOREIGN KEY#(三大键)

3、删除表级约束
alter table 表名 drop PRIMARY KEY/INDEX/FOREIGN KEY#(三大键)
```

```mysql
#1.添加非空约束
ALTER TABLE stuinfo MODIFY COLUMN stuname VARCHAR(20)  NOT NULL;

#2.添加默认约束
ALTER TABLE stuinfo MODIFY COLUMN age INT DEFAULT 18;

#3.添加主键
    #①列级约束
    ALTER TABLE stuinfo MODIFY COLUMN id INT PRIMARY KEY;
    #②表级约束
    ALTER TABLE stuinfo ADD PRIMARY KEY(id);

#4.添加唯一
    #①列级约束
    ALTER TABLE stuinfo MODIFY COLUMN seat INT UNIQUE;
    #②表级约束
    ALTER TABLE stuinfo ADD UNIQUE(seat);

#5.添加外键
ALTER TABLE stuinfo ADD CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id); 

#6.删除非空约束
ALTER TABLE stuinfo MODIFY COLUMN stuname VARCHAR(20) NULL;

#7.删除默认约束
ALTER TABLE stuinfo MODIFY COLUMN age INT ;

#8.删除主键
ALTER TABLE stuinfo DROP PRIMARY KEY;

#9.删除唯一
ALTER TABLE stuinfo DROP INDEX seat;

#10.删除外键
ALTER TABLE stuinfo DROP FOREIGN KEY fk_stuinfo_major;
```

#### 自增长列

AUTO INCREMENT是除了六大约束外的另一个重要约束，可以不用手动设置该值，系统默认往后自增

- 一个表至多有一个自增长列
- 自增长列必须为一个key(主键、唯一键)

```mysql
#一、创建表时设置自增长列
CREATE TABLE tab_identity(
	id INT  ,
	NAME FLOAT UNIQUE AUTO_INCREMENT,
	seat INT 
) TRUNCATE TABLE tab_identity;

INSERT INTO tab_identity(id,NAME) VALUES(NULL,'john');

INSERT INTO tab_identity(NAME) VALUES('lucy');
```

