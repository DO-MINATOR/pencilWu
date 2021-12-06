### 简介

生产环境中，性能安全性都需要提升几个数量级，因此需要搭建一套主从架构，实现容灾备份，读写分离，分库分表。

主从集群的优势：

- 数据安全：为数据添加备份，互主架构也可以维护数据的安全性。
- 读写分离：读多写少的场景，读请求可以由专门的一个服务器来应答。
- 故障转移：主服务器宕机后，由从服务器继续提供服务，通过MMM、MHA、MGR。

**同步原理：**

 MySQL服务的主从架构一般都是通过binlog日志文件来进行的。即在主服务上打开binlog记录每一步的数据库操作，然后从服务上会有一个IO线程，负责跟主服务建立一个TCP连接，请求主服务将binlog传输过来。这时，主库上会有一个IO dump线程，负责通过这个TCP连接把Binlog日志传输给从库的IO线程。接着从服务的IO线程会把读取到的binlog日志数据写入自己的relay日志文件中。然后从服务上另外一个SQL线程会读取relay日志里的内容，进行操作重演，达到还原数据的目的。我们通常对MySQL做的读写分离配置就必须基于主从架构来搭建。

![image-20211206100223766](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211206100223766.png)

binlog日志也可以作为其他中间件的缓存对象，如redis。

### 主从搭建

#### 主库

首先，配置主节点的mysql配置文件： /etc/my.cnf 这一步需要对master进行配置，主要是需要打开binlog日志，以及指定severId。我们打开MySQL主服务的my.cnf文件，在文件中一行server-id以及一个关闭域名解析的配置。然后重启服务。

```cnf
[mysqld]
#服务节点的唯一标识。需要给集群中的每个服务分配一个单独的ID。
server-id=47
#开启binlog
log_bin=master-bin
log_bin-index=master-bin.index
skip-name-resolve
# 设置连接端口
port=3306
# 设置mysql的安装目录
basedir=/usr/local/mysql
# 设置mysql数据库的数据的存放目录
datadir=/usr/local/mysql/mysql-files
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
#mysql_native_password
default_authentication_plugin=mysql_native_password
```

服务节点的唯一标识。需要给集群中的每个服务分配一个单独的ID。

```mysql
mysql -u root -p
GRANT REPLICATION SLAVE ON *.* TO 'root'@'%';
flush privileges;
#查看主节点同步状态：
show master status;
```

![image-20211206100617332](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211206100617332.png)

这个指令结果中的File和Position记录的是当前日志的binlog文件以及文件中的索引。 而后面的Binlog_Do_DB和Binlog_Ignore_DB这两个字段是表示需要记录binlog文件的库以及不需要记录binlog文件的库。目前我们没有进行配置，就表示是针对全库记录日志。这两个字段如何进行配置，会在后面进行介绍。

开启binlog后，数据库中的所有操作都会被记录到datadir当中，以一组轮询文件的方式循环记录。而指令查到的File和Position就是当前日志的文件和位置。而在后面配置从服务时，就需要通过这个File和Position通知从服务从哪个地方开始记录binLog。

#### 从库

```cnf
[mysqld]
#主库和从库需要不一致
server-id=48
#打开MySQL中继日志
relay-log-index=slave-relay-bin.index
relay-log=slave-relay-bin
#打开从服务二进制日志
log-bin=mysql-bin
#使得更新的数据写进二进制日志中
log-slave-updates=1
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=/usr/local/mysql
# 设置mysql数据库的数据的存放目录
datadir=/usr/local/mysql/mysql-files
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
#mysql_native_password
default_authentication_plugin=mysql_native_password
```

然后我们启动mysqls的服务，并设置他的主节点同步状态。

```mysql
#登录从服务
mysql -u root -p;
#设置同步主节点：
CHANGE MASTER TO
MASTER_HOST='192.168.232.128',
MASTER_PORT=3306,
MASTER_USER='root',
MASTER_PASSWORD='root',
MASTER_LOG_FILE='master-bin.000004',
MASTER_LOG_POS=156
GET_MASTER_PUBLIC_KEY=1;
#开启slave
start slave;
#查看主从同步状态
show slave status;
```

![image-20211206100924333](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211206100924333.png)

#### 主从测试

测试时，我们先用showdatabases，查看下两个MySQL服务中的数据库情况

![image-20211206101001171](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211206101001171.png)

在主服务器上创建一个数据库

```mysql
mysql> create database syncdemo;
Query OK, 1 row affected (0.00 sec)
```

![image-20211206101058641](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211206101058641.png)

创建表，并插入数据

```mysql
mysql> use syncdemo;
Database changed
mysql> create table demoTable(id int not null);
Query OK, 0 rows affected (0.02 sec)

mysql> insert into demoTable value(1);
Query OK, 1 row affected (0.01 sec)
```

主从库都有了该数据。

另外，这个主从架构是有可能失败的，如果在slave从服务上查看slave状态，发现Slave_SQL_Running=no，就表示主从同步失败了。这有可能是因为在从数据库上进行了写操作，与同步过来的SQL操作冲突了，也有可能是slave从服务重启后有事务回滚了。

出现同步失败后，需要停止salve，并重新设置主库，再启动。

```mysql
mysql> stop slave ;
mysql> change master to .....
mysql> start slave ;
```

### 集群扩展

#### 部分同步/非同步

 之前提到，我们目前配置的主从同步是针对全库配置的，而实际环境中，一般并不需要针对全库做备份，而只需要对一些特别重要的库或者表来进行同步。那如何针对库和表做同步配置呢？

首先在Master端：在my.cnf中，可以通过以下这些属性指定需要针对哪些库或者哪些表记录binlog

```conf
#需要同步的二进制数据库名
binlog-do-db=masterdemo
#只保留7天的二进制日志，以防磁盘被日志占满(可选)
expire-logs-days  = 7
#不备份的数据库
binlog-ignore-db=information_schema
binlog-ignore-db=performation_schema
binlog-ignore-db=sys
```

然后在Slave端：在my.cnf中，需要配置备份库与主服务的库的对应关系。

```conf
#如果salve库名称与master库名相同，使用本配置
replicate-do-db = masterdemo 
#如果master库名[mastdemo]与salve库名[mastdemo01]不同，使用以下配置[需要做映射]
replicate-rewrite-db = masterdemo -> masterdemo01
#如果不是要全部同步[默认全部同步]，则指定需要同步的表
replicate-wild-do-table=masterdemo01.t_dict
replicate-wild-do-table=masterdemo01.t_num
```

#### 读写分离配置

我们要注意，目前我们的这个MySQL主从集群是单向的，也就是只能从主服务同步到从服务，而从服务的数据表更是无法同步到主服务的。 所以，在这种架构下，为了保证数据一致，通常会需要保证数据只在主服务上写，而从服务只进行数据读取。这个功能，就是大名鼎鼎的读写分离。但是这里要注意下，mysql主从本身是无法提供读写分离的服务的，需要由业务自己来实现。这也是我们后面要学的ShardingSphere的一个重要功能。

到这里可以看到，在MySQL主从架构中，是需要严格限制从服务的数据写入的，一旦从服务有数据写入，就会造成数据不一致。并且从服务在执行事务期间还很容易造成数据同步失败。

如果需要限制用户写数据，我们可以在从服务中将read_only参数的值设为1`set global read_only=1;` 这样就可以限制用户写入数据。但是这个属性有两个需要注意的地方：

1. read_only=1设置的只读模式，不会影响slave同步复制的功能。 所以在MySQL slave库中设定了read_only=1后，通过 "show slave status\G" 命令查看salve状态，可以看到salve仍然会读取master上的日志，并且在slave库中应用日志，保证主从数据库同步一致；
2. read_only=1设置的只读模式， 限定的是普通用户进行数据修改的操作，但不会限定具有super权限的用户的数据修改操作。 在MySQL中设置read_only=1后，普通的应用用户进行insert、update、delete等会产生数据变化的DML操作时，都会报出数据库处于只读模式不能发生数据变化的错误，但具有super权限的用户，例如在本地或远程通过root用户登录到数据库，还是可以进行数据变化的DML操作； 如果需要限定super权限的用户写数据，可以设置super_read_only=0。

#### 复杂集群架构

一主一从只是最基础的架构，根据业务，还会拓展出复杂庞大的集群。如一主多从，级联多从，多主，互主（互主只需要在salve上也开启binlog配置），环形互主。

### GTID主从同步

 上面我们搭建的集群方式，是基于Binlog日志记录点的方式来搭建的，这也是最为传统的MySQL集群搭建方式。而在这个实验中，可以看到有一个Executed_Grid_Set列，暂时还没有用上。实际上，这就是另外一种搭建主从同步的方式，即GTID搭建方式。这种模式是从MySQL5.6版本引入的。

 GTID的本质也是基于Binlog来实现主从同步，只是他会基于一个全局的事务ID来标识同步进度。GTID即全局事务ID，全局唯一并且趋势递增，他可以保证为每一个在主节点上提交的事务在复制集群中可以生成一个唯一的ID 。

 在基于GTID的复制中，首先从服务器会告诉主服务器已经在从服务器执行完了哪些事务的GTID值，然后主库会有把所有没有在从库上执行的事务，发送到从库上进行执行，并且使用GTID的复制可以保证同一个事务只在指定的从库上执行一次，这样可以避免由于偏移量的问题造成数据不一致。

### 集群中期扩容

我们现在已经搭建成功了一主一从的MySQL集群架构，那要扩展到一主多从的集群架构，其实就比较简单了，只需要增加一个binlog复制就行了。

 但是如果我们的集群是已经运行过一段时间，这时候如果要扩展新的从节点就有一个问题，之前的数据没办法从binlog来恢复了。这时候在扩展新的slave节点时，就需要增加一个数据复制的操作。

 MySQL的数据备份恢复操作相对比较简单，可以通过SQL语句直接来完成。具体操作可以使用mysql的bin目录下的mysqldump工具。

```mysql
mysqldump -u root -p --all-databases > backup.sql
```

 通过这个指令，就可以将整个数据库的所有数据导出成backup.sql，然后把这个backup.sql分发到新的MySQL服务器上，并执行下面的指令将数据全部导入到新的MySQL服务中。

```mysql
mysql -u root -p < backup.sql
```

### 半同步复制

采用异步复制机制可能会导致数据丢失。因为主库收到写请求后直接响应客户端，而不管从库的状态。

![image-20211206103210119](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211206103210119.png)

而如果采用同步复制，又会导致线程积压。因此，采用带超时机制的半同步复制。 半同步复制机制是一种介于异步复制和全同步复制之前的机制。主库在执行完客户端提交的事务后，并不是立即返回客户端响应，而是等待至少一个从库接收并写到relay log中，才会返回给客户端。MySQL在等待确认时，默认会等10秒，如果超过10秒没有收到ack，就会降级成为异步复制。

![image-20211206103351898](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211206103351898.png)

 这种半同步复制相比异步复制，能够有效的提高数据的安全性。但是这种安全性也不是绝对的，他只保证事务提交后的binlog至少传输到了一个从库，并且并不保证从库应用这个事务的binlog是成功的。另一方面，半同步复制机制也会造成一定程度的延迟，这个延迟时间最少是一个TCP/IP请求往返的时间。整个服务的性能是会有所下降的。而当从服务出现问题时，主服务需要等待的时间就会更长，要等到从服务的服务恢复或者请求超时才能给用户响应。

#### 半同步搭建

 半同步复制需要基于特定的扩展模块来实现。而mysql从5.5版本开始，往上的版本都默认自带了这个模块。这个模块包含在mysql安装目录下的lib/plugin目录下的semisync_master.so和semisync_slave.so两个文件中。需要在主服务上安装semisync_master模块，在从服务上安装semisync_slave模块。

![image-20211206103644138](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211206103644138.png)

首先我们登陆主服务，安装semisync_master模块：

```mysql
mysql> install plugin rpl_semi_sync_master soname 'semisync_master.so';
Query OK, 0 rows affected (0.01 sec)

mysql> show global variables like 'rpl_semi%';
+-------------------------------------------+------------+
| Variable_name                             | Value      |
+-------------------------------------------+------------+
| rpl_semi_sync_master_enabled              | OFF        |
| rpl_semi_sync_master_timeout              | 10000      |
| rpl_semi_sync_master_trace_level          | 32         |
| rpl_semi_sync_master_wait_for_slave_count | 1          |
| rpl_semi_sync_master_wait_no_slave        | ON         |
| rpl_semi_sync_master_wait_point           | AFTER_SYNC |
+-------------------------------------------+------------+
6 rows in set, 1 warning (0.02 sec)

mysql> set global rpl_semi_sync_master_enabled=ON;
Query OK, 0 rows affected (0.00 sec)
```

- 第一行是通过扩展库来安装半同步复制模块，需要指定扩展库的文件名。
- 第二行查看系统全局参数，rpl_semi_sync_master_timeout就是半同步复制时等待应答的最长等待时间，默认是10秒，可以根据情况自行调整。
- 第三行则是打开半同步复制的开关。

然后我们登陆从服务，安装smeisync_slave模块：

```mysql
mysql> install plugin rpl_semi_sync_slave soname 'semisync_slave.so';
Query OK, 0 rows affected (0.01 sec)

mysql> show global variables like 'rpl_semi%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| rpl_semi_sync_slave_enabled     | OFF   |
| rpl_semi_sync_slave_trace_level | 32    |
+---------------------------------+-------+
2 rows in set, 1 warning (0.01 sec)

mysql> set global rpl_semi_sync_slave_enabled = on;
Query OK, 0 rows affected (0.00 sec)

mysql> show global variables like 'rpl_semi%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| rpl_semi_sync_slave_enabled     | ON    |
| rpl_semi_sync_slave_trace_level | 32    |
+---------------------------------+-------+
2 rows in set, 1 warning (0.00 sec)

mysql> stop slave;
Query OK, 0 rows affected (0.01 sec)

mysql> start slave;
Query OK, 0 rows affected (0.01 sec)
```

### 同步延迟

 在我们搭建的这个主从集群中，有一个比较隐藏的问题，就是这样的主从复制之间会有延迟。这在做了读写分离后，会更容易体现出来。即数据往主服务写，而读数据在从服务读。这时候这个主从复制延迟就有可能造成刚插入了数据但是查不到。当然，这在我们目前的这个集群中是很难出现的，但是在大型集群中会很容易出现。

 出现这个问题的根本在于：面向业务的主服务数据都是多线程并发写入的，而从服务是单个线程慢慢拉取binlog，这中间就会有个效率差。所以解决这个问题的关键是要让从服务也用多线程并行复制binlog数据。

 MySQL自5.7版本后就已经支持并行复制了。可以在从服务上设置slave_parallel_workers为一个大于0的数，然后把slave_parallel_type参数设置为LOGICAL_CLOCK，这就可以了。

我们之前的MySQL服务集群，都是使用MySQL自身的功能来搭建的集群。但是这样的集群，不具备高可用的功能。即如果是MySQL主服务挂了，从服务是没办法自动切换成主服务的。而如果要实现MySQL的高可用，需要借助一些第三方工具来实现。

 常见的MySQL集群方案有三种: MMM、MHA、MGR。这三种高可用框架都有一些共同点：

- 对主从复制集群中的Master节点进行监控
- 自动的对Master进行迁移，通过VIP。
- 重新配置集群中的其它slave对新的Master进行同步