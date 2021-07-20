### MySQL组件架构

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/12570" alt="dsa" style="zoom:80%;" />

#### Server

连接器、查询缓存、分析器、优化器、执行器、内置函数以及一切跨存储引擎的功能都在该层实现，如存储过程、触发器、视图。

#### Store

可插拔式的存储引擎，负责数据存储和提取，常用的innodb为MySQL5.5的默认引擎，特性为聚集索引，行级锁等。

### 连接器

负责与客户端建立链接，权限检查，维持和管理连接。

1. 如果用户名或密码不对，你就会收到一个"Access denied for user"的错误，然后客户端程序结束执行。 
2. 如果用户名密码认证通过，连接器会到权限表里面查出你拥有的权限。之后，这个连接里面的权限判断逻辑，都将依赖于此时读到的权限。

如果此时已经建立了连接，即使修改了该用户的在User表下的权限，也无法立即生效，因为连接器维持的是在内存中的权限数据，防止高并发场景下的性能故障。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/12637" alt="dsa" style="zoom: 67%;" />

修改root用户密码

```mysql
mysql> CREATE USER 'username'@'host' IDENTIFIED BY 'password'; //创建新用户
mysql> grant all privileges on *.* to 'username'@'%'; //赋权限,%表示所有(host)
mysql> flush privileges //刷新数据库
mysql> update user set password=password(”123456″) where user=’root’;(设置用户名密码)
mysql> show grants for root@"%"; 查看当前用户的权限
```

show processlist查看当前所有连接的状态，如果time达到28800（8h），就自动断开。

![dsa](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/12632)

```mysql
mysql> show global variables like "wait_timeout";
mysql> set global wait_timeout=28800; 设置全局服务器关闭非交互连接之前等待活动的秒数
```

![sda](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/12654)

同样，MySQL也支持长连接，多次查询共用一个socket，为了避免OOM，执行过一段时间查询后，或者一次大查询后，手动执行`mysql_reset_connection`来初始化链接资源。

### 查询缓存

在早期版本中，支持查询缓存，MySQL每次收到一个查询SQL语句后，会在缓存中查找是否有对应的结果集。之所以该功能后被舍弃，是因为缓存经常会失效，CUD操作会导致前后查询的结果不同，就需要重新缓存，因此该功能默认关闭，只有对一些静态表（长时间不会改动的表）开启查询缓存。

**mysql8.0已经移除了查询缓存功能**

```cnf
//my.cnf文件
#query_cache_type有3个值 0代表关闭查询缓存OFF，1代表开启ON，2（DEMAND）代表当sql语句中有SQL_CACHE关键词时才缓存
query_cache_type=2
```

```mysql
 select SQL_CACHE * from test where ID=5；
 #指定查询时使用SQL_CACHE，如果没查到，将本次结果进行缓存
```

```mysql
 show status like'%Qcache%'; //查看运行的缓存信息
```

![ds](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/12798)

- Qcache_free_blocks:表示查询缓存中目前还有多少剩余的blocks，如果该值显示较大，则说明查询缓存中的内存碎片过多了，可能在一定的时间进行整理。
- Qcache_free_memory:查询缓存的内存大小，通过这个参数可以很清晰的知道当前系统的查询内存是否够用，是多了，还是不够用，DBA可以根据实际情况做出调整。
- Qcache_hits:表示有多少次命中缓存。我们主要可以通过该值来验证我们的查询缓存的效果。数字越大，缓存效果越理想。
- Qcache_inserts: 表示多少次未命中然后插入，意思是新来的SQL请求在缓存中未找到，不得不执行查询处理，执行查询处理后把结果insert到查询缓存中。这样的情况的次数，次数越多，表示查询缓存应用到的比较少，效果也就不理想。当然系统刚启动后，查询缓存是空的，这很正常。
- Qcache_lowmem_prunes:该参数记录有多少条查询因为内存不足而被移除出查询缓存。通过这个值，用户可以适当的调整缓存大小。
- Qcache_not_cached: 表示因为query_cache_type的设置而没有被缓存的查询数量。
- Qcache_queries_in_cache:当前缓存中缓存的查询数量。
- Qcache_total_blocks:当前缓存的block数量。

### 分析器

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/12742" alt="dsad" style="zoom:67%;" />

<img src="https://note.youdao.com/yws/public/resource/6480d1e092ed1c14a53d86cd66a73139/xmlnote/363B9FBC8A3545F08F02C9231983B1E9/12751" alt="dsa" style="zoom: 50%;" />

### 优化器

在正式执行SQL语句前，分析possible_keys，cost，是否采取索引，采取哪个索引等。

```mysql
select * from test1 join test2 using(ID) where test1.name=yangguo and test2.name=xiaolongnv;
```

- 先从表 test1 里面取出 name=yangguo的记录的 ID 值，再根据 ID 值关联到表 test2，再判断 test2 里面 name的值是否等于 yangguo。
- 先从表 test2 里面取出 name=xiaolongnv 的记录的 ID 值，再根据 ID 值关联到 test1，再判断 test1 里面 name 的值是否等于 yangguo。

这两种执行方法的逻辑结果是一样的，但是执行的效率会有不同，而优化器的作用就是决定选择使用哪一个方案。

### 执行器

根据引擎提供的接口，开始执行SQL语句。

### bin-log归档

binlog是Server层提供的二进制日志记录，记录了所有CUD操作。

- 逻辑日志，记录的是执行逻辑
- 每次追加写入

因此，如果根据日志进行数据复原。

```cnf
//my.cnf配置开启binlog
log-bin=/usr/local/mysql/data/binlog/mysql-bin
#注意5.7以及更高版本需要配置本项：server-id=123454（自定义,保证唯一性）;

binlog-format=ROW
#binlog格式，有3种statement,row,mixed

sync-binlog=1
#表示每1次执行写入就与硬盘同步，会影响性能，为0时表示，事务提交时mysql不做刷盘操作，由系统决定
```

```mysql
mysql> show variables like '%log_bin%'; 查看bin-log是否开启
mysql> flush logs; 会多一个最新的bin-log日志
mysql> show master status; 查看最后一个bin-log日志的相关信息
mysql> reset master; 清空所有的bin-log日志

mysql> /usr/local/mysql/bin/mysqlbinlog --no-defaults /usr/local/mysql/data/binlog/mysql-bin.000001 
#查看binlog内容
```

binlog里的内容不具备可读性，所以需要我们自己去判断恢复的逻辑点位，怎么观察呢？看重点信息，比如begin,commit这种关键词信息，只要在binlog当中看到了，你就可以理解为begin-commit之间的信息是一个完整的事务逻辑,然后再根据位置position判断恢复即可。binlog内容如下：

![d](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/12847)

执行归档

```mysql
#恢复全部数据
/usr/local/mysql/bin/mysqlbinlog --no-defaults /usr/local/mysql/data/binlog/mysql-bin.000001 
| mysql -uroot -p tuling(数据库名)
#恢复指定位置数据
/usr/local/mysql/bin/mysqlbinlog --no-defaults /usr/local/mysql/data/binlog/mysql-bin.000001 --start-position="408" --stop-position="731" | mysql -uroot -p tuling(数据库)
#恢复指定时间段数据
/usr/local/mysql/bin/mysqlbinlog --no-defaults /usr/local/mysql/data/binlog/mysql-bin.000001  --start-date= "2019-03-02 11:55:00" --stop-date= "2018-03-02 12:00:00" | mysql -uroot -p test(数据库)
```

