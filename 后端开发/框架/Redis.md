### 简介

- 功能：Java、Jsp、Tomcat、JDBC、MySQL
- 扩展：Spring、SpringMvc、Mybatis
- 性能：NoSQL、多线程、Nginx、MQ、RPC

web1.0时代单点服务器即可解决用户需求，web2.0用户数量提升，做扩容、负载均衡、分布式、集群等，又会导致新的问题，如session复制导致的冗余，如果数据库也要做拆分又会导致业务逻辑破坏。因此提出缓存数据库技术。

NoSQL，意即“不仅仅是SQL”，泛指非关系型数据库，以key-value的形式存储。

- 不遵循SQL标准
- 不支持ACID事务
- 高性能

适用于

- 海量数据场景
- 高并发读写
- 非结构化数据
- 高频次、热门数据访问

![image-20210517193630745](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210517193630745.png)

不适用于

- 需要事务支持的场景
- 结构化查询，需要用到即席查询

### 安装配置

key-value中value支持的数据类型多样，包括String、list、set、hash、zset。支持push/pop、add/remove操作，且都是原子性的，支持数据的持久化。支持主从同步。

源码安装，通过make install编译安装。安装目录位于/usr/local/bin，后台启动服务，默认端口6379。

redis采用单线程+IO复用技术

关闭服务命令：`redis-cli shutdown`

**redis.config**

- 默认情况bind=127.0.0.1只能接受本机的访问请求
- protected-mode：将本机访问保护模式设置no
- port：默认端口号6379
- daemonize：是否设置为守护进程
- maxmemory：最大内存空间，如果超出内存空间，且不允许移除或没有设置移除规则，则报错。
- maxmemory-policy：移除规则
  1. volatile-lru：使用LRU算法移除key，只对设置了过期时间的键；（最近最少使用）
  2. allkeys-lru：在所有集合key中，使用LRU算法移除key
  3. volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键
  4. allkeys-random：在所有集合key中，移除随机的key
  5. volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
  6. noeviction：不进行移除。针对写操作，只是返回错误信息
- maxmemory-samples：LRU和最小ttl算法非精确算法，因此只会检查samples个样本。

### 基础命令

- 切换数据库：`select id`
- 查看当前数据库key数量：`dbsize`
- 清空当前库：`flushdb`
- 清空全部库：`flushall`
- `keys *`查看当前库所有key
- `exists key`判断某个key是否存在
- `type key` 查看你的key是什么类型
- `del key`    删除指定的key数据
- `unlink key`  根据value选择非阻塞删除
- `expire key 10`  10秒钟：为给定的key设置过期时间
- `ttl key` 查看还有多少秒过期，-1表示永不过期，-2表示已过期

### 五大数据类型

#### String

最基础的数据类型，可以存放二进制数据，数据结构为简单动态字符串，类似于ArrayList，采用动态扩容方式，但大小不超过512M。

- set  \<key>\<value>添加键值对
- *NX：当数据库中key不存在时，可以将key-value添加数据库
- *XX：当数据库中key存在时，可以将key-value添加数据库，与NX参数互斥
- *EX：key的超时秒数
- *PX：key的超时毫秒数，与EX互斥
- get  \<key>查询对应键值
- append \<key>\<value>将给定的\<value> 追加到原值的末尾
- strlen \<key>获得值的长度
- setnx \<key>\<value>只有在 key 不存在时  设置 key 的值
- incr \<key>
- 将 key 中储存的数字值增1
- 只能对数字值操作，如果为空，新增值为1
- decr \<key>
- 将 key 中储存的数字值减1
- 只能对数字值操作，如果为空，新增值为-1
- incrby / decrby \<key><步长>将 key 中储存的数字值增减。自定义步长。
- getrange \<key><起始位置><结束位置>
- setrange \<key><起始位置>\<value>
- getset \<key>\<value>以新换旧，设置了新值同时获得旧值。

#### List

key-value中的value类型为list，实际上是双向链表。且当list元素较少时会用连续内存分配ziplist，并没有前后指针，当元素个数增多时，才会采取多个ziplist合并为一个quicklist。

![image-20210517201223975](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210517201223975.png)

- lpush/rpush \<key>\<value1>\<value2>\<value3> .... 从左边/右边插入一个或多个值。
- lpop/rpop \<key>从左边/右边吐出一个值。值在键在，值光键亡。
- rpoplpush \<key1>\<key2>从\<key1>列表右边吐出一个值，插到\<key2>列表左边。
- lrange \<key>\<start>\<stop>
- 按照索引下标获得元素(从左到右)
- lrange mylist 0 -1  0左边第一个，-1右边第一个，（0-1表示获取所有）
- lindex \<key>\<index>按照索引下标获得元素(从左到右)
- llen \<key>获得列表长度 
- linsert \<key> before \<value>\<newvalue>在\<value>的后面插入\<newvalue>插入值
- lrem \<key>\<n>\<value>从左边删除n个value(从左到右)
- lset\<key>\<index>\<value>将列表key下标为index的值替换成value

#### Set

value是set类型，相比list，可以自动去重，底层实际上是value为null的hash表，时间复杂度为O(1)。

- sadd \<key>\<value1>\<value2> ..... 
- 将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略
- smembers \<key>取出该集合的所有值。
- sismember \<key>\<value>判断集合\<key>是否为含有该\<value>值，有1，没有0
- scard\<key>返回该集合的元素个数。
- srem \<key>\<value1>\<value2> .... 删除集合中的某个元素。
- spop \<key>随机从该集合中吐出一个值。
- srandmember \<key>\<n>随机从该集合中取出n个值。不会从集合中删除 。
- smove \<source>\<destination>value把集合中一个值从一个集合移动到另一个集合
- sinter \<key1>\<key2>返回两个集合的交集元素。
- sunion \<key1>\<key2>返回两个集合的并集元素。
- sdiff \<key1>\<key2>返回两个集合的**差集**元素(key1中的，不包含key2中的)

#### Hash

value是hashmap，存储对象类型。同样，当存储数量较少时，使用ziplist（压缩列表），map对象多时，改为hashtable。

- hset \<key>\<field>\<value>给\<key>集合中的 \<field>键赋值\<value>
- hget \<key1>\<field>从\<key1>集合\<field>取出 value 
- hmset \<key1>\<field1>\<value1>\<field2>\<value2>... 批量设置hash的值
- hexists\<key1>\<field>查看哈希表 key 中，给定域 field 是否存在。 
- hkeys \<key>列出该hash集合的所有field
- hvals \<key>列出该hash集合的所有value
- hincrby \<key>\<field>\<increment>为哈希表 key 中的域 field 的值加上增量 1  -1
- hsetnx \<key>\<field>\<value>将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 

#### Zset

value是map\<String,Double>，根据权重值进行排序，通过跳表可以加速查询数据。

![image-20210518093545521](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210518093545521.png)

- zadd \<key>\<score1>\<value1>\<score2>\<value2>…
- 将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
- zrange \<key>\<start>\<stop> [WITHSCORES]  
- 返回有序集 key 中，下标在\<start>\<stop>之间的元素
- 带WITHSCORES，可以让分数一起和值返回到结果集。
- zrangebyscore key minmax [withscores] [limit offset count]
- 返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。 
- zrevrangebyscore key maxmin [withscores] [limit offset count]        
- 同上，改为从大到小排列。 
- zincrby \<key>\<increment>\<value>   为元素的score加上增量
- zrem \<key>\<value>删除该集合下，指定值的元素
- zcount \<key>\<min>\<max>统计该集合，分数区间内的元素个数 
- zrank \<key>\<value>返回该值在集合中的排名，从0开始。

### 发布订阅

消息通信模式

- 订阅者：subscribe channel1
- 发布者：publish channel1 hello

### Jedis

java操纵redis的包

```java
Jedis jedis = new Jedis("192.168.137.3",6379);
String pong = jedis.ping();
System.out.println("连接成功："+pong);
```

### 事务

Redis事务是一个单独的隔离操作，事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。这样串联多个命令，防止别的命令来插队。

- Mutli：创建队列，但不会立即执行。出错则全部不执行。
- discard：销毁队列。
- exec：执行队列。一个错误不会影响其他的执行。

三特性：

- 单独的隔离操作：所有命令序列化、按需执行
- 没有隔离级别
- 没有原子性，执行出错不会影响别的命令

但是不同事务中的命令有可能会冲突，例如对同一个value进行增减操作。

由此引出锁机制

### 乐观/悲观锁

- 悲观锁：每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。
- 乐观锁：每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号机制(版本号的修改是序列化的)。乐观锁适用于多读的应用类型，这样可以提高吞吐量。Redis就是利用这种check-and-set机制实现事务的。

在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

秒杀系统中利用redis可能存在的问题及解决方案：

- 连接超时，使用连接池
- 超卖问题，使用乐观锁
- 乐观锁导致的库存遗留，通过循环或者lua脚本

LUA可将复杂的redis操作写为一个脚本，一次性提交给redis执行，减少redis反复连接，同时lua脚本具有一定的原子性，不会被打断，因此可以保证不会超卖和库存遗留问题。

### 持久化

#### RDB

在指定时间间隔内将该部分变动数据及以前数据持久化到磁盘中，持久化时，单独fork一个子进程将数据写入到临时文件中，完毕后在替换。保存位置默认为启动路径。

恢复机制是在每次启动redis-server时将rdb.dump反序列化。

优势：

1. 适合大规模数据恢复
2. 节省磁盘空间
3. 恢复速度快

劣势：

存在数据不完整性，因为是在指定时间段内保存变化的数据。

#### AOF

增量保存，通过日志方式记录变动的数据，恢复时重新执行一边所有变动记录。当AOF文件大小超过一定值时，进行rewrite重写，以缩减容量。

AOF每次持久化时，都会将记录直接追加到文件末尾，只有当进行重写时，才会进行写时复制技术，即fork出新进程，将RDB数据保存到新文件中，在同时追加记录到两个文件中，之后再将新文件替换旧文件，完成容量压缩。

优势：

1. 数据更加完整
2. 中间的记录过程可以辅助调查

劣势：

1. 占用更多的磁盘空间
2. 恢复速度较慢
3. 性能较差

如果二者同时启用，则只会读取AOF文件，因为更加完整。只有当需要快速重启或者时间点保存时，RDB才有用。

写时复制技术有可能导致短暂的响应延迟，因此建议Master采取AOF，Slave采取RDB。

### 主从复制

Master写，Slave读。通过读写分离，实现性能扩展，以及容灾的快速恢复（Salve因为有多个，因此读服务器可以自动完成切换，而Master只有一个，因此主机如果想要实现容灾备份的话，需要集群）

- 创建多个redis-6379.conf、redis-6380.conf、redis-6381.conf
  - include /redis/redis.conf，引入完整配置
  - pidfile /var/run/redis_6379.pid
  - port 6379
  - dbfilename dump6379.rdb
- 启动三服务，redis-server redis-6379.conf
- info replication查看服务器属性
- slaveof 127.0.0.1 6379，在6380、6381上配置从属信息

#### 一主二仆

- 主动复制：当salve初次连接到master时，会进行主动的全量更新（slave挂掉重启后依然可以完成）
- 被动传输：在master每次更新数据后，将增量信息主动发送给slave

#### 薪火相传

![image-20210528110151054](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210528110151054.png)

中间的slave既是从机又是主机，后面的从机通过slaveof命令配置中间节点为其master，重新连接时，将进行新的全量更新。

#### 反客为主

slaveof no one，升级为主机，一般在Master断掉后，自选一台Slave晋升为Master，其余Slave连接上该Master。

#### 复制原理

- Slave每次在连接的第一次时，都会发送sync命令给服务器，请求全量更新。
- 后续的修改操作，Master会发送增量信息给Slave。

### 哨兵模式

反客为主的自动化实现，通过配置哨兵，自动监测Master是否宕机，如果宕机，则按照一定规则选举一台Slave成为Master，之后其余Slave以及重启后的旧的Master将变为Slaves。

- 配置sentinel.conf，添加命令`sentinel monitor mymaster 127.0.0.1 6379 1`，mymaster为哨兵的名称，1为至少需要多少个哨兵同意的数量。
- 启动是哨兵，`redis-sentinel ./sentinel.conf`

选举规则是通过

1. 优先级选取的，在redis.conf中的replic-priority上配置，数字越低优先级越高
2. 偏移量越大越好，（获取数据最全的）
3. runid越小越好，随机生成。

```java
static Jedis getFromPool() {
    Set<String> sentinelSet = new HashSet<String>();
    sentinelSet.add("192.168.137.181:26379");
    JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
    jedisPoolConfig.setMaxTotal(20);
    jedisPoolConfig.setBlockWhenExhausted(true);
    jedisPoolConfig.setMaxWaitMillis(2000);
    jedisPoolConfig.setTestOnBorrow(true);
    jedisSentinelPool = new JedisSentinelPool("mymaster", sentinelSet, jedisPoolConfig);
    Jedis resource = jedisSentinelPool.getResource();
    return resource;
}
```

通过sentinel（哨兵服务）获取jedis连接对象，这样Master挂掉后，又会重新连接上新的Master。

### 集群

解决生产环境中的容量、写操作压力分担。通过无中心化集群，在连接集群服务时，可自动将请求打在不同服务器上。

- 水平扩容，如果有N个Master，则每个Master负责总数据量1/N的业务量。
- 分区备份，Master和对应的Slave会分配在不同的IP下，防止整个挂掉。

这里以6个服务为例，6379、6380、6381、6389、6390、6391。

redis-6379.conf

```conf
include /redis/redis.conf
port 6379
pidfile "/var/run/redis_6379.pid"
dbfilename "dump6379.rdb"
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
```

redis-server启动6个服务后，再通过集群方式进行管理。在redis安装目录下通过

`redis-cli --cluster create --cluster-replicas 1 192.168.224.1:6379 192.168.224.1:6380 192.168.224.1:6381 192.168.224.1:6389 192.168.224.1:6390 192.168.224.1:6391`

![image-20210531152608541](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210531152609585.png)

客户端连接方式如果采用以前的单点登录，则会发生操作转移，因此建议使用集群登录命令，`redis-cli -c -p 6379`，这样，如果遇到数据需要在别的机器上进行写操作时，则会自动切换到对应主机。

#### 插槽Slot

一个 Redis 集群包含 16384 个插槽（hash slot）， 数据库中的每个键都属于这 16384 个插槽的其中一个， 集群使用公式 CRC16(key) % 16384 来计算键 key 属于哪个槽。每个slot可能发生“hash”冲突。

```txt
cluster keyslot cust //返回key cust所在的slot号
cluster countkeysinslot 4847 //返回4847号slot中拥有的keys的数量
cluster getkeysinslot 4847 10 //返回4847号slot中10个key
```

**注意：**上述命令需要slot段对应的节点上(Master/Slave)执行才有效

#### 故障恢复

如果Master挂掉后，对应的Slave会自动晋升为Master，原来的Master恢复后，自动变为Slave。如果某一段slot的Master和Slave都挂掉后，会根据cluster-require-full-coverage对应的配置是否为yes决定集群服务是否还能运行，如果还能运行，则当前数据段无法工作。

#### Jedis集群

```java
@Test
public void test01() {
    HostAndPort hostAndPort = new HostAndPort("192.168.137.181", 6379);//任意节点都可
    JedisCluster jedisCluster = new JedisCluster(hostAndPort);
    String k1 = jedisCluster.get("k1");
    System.out.println(k1);
}
```

无中心化集群连接方式。

### 其他问题

#### 缓存穿透

大量（非法）请求打在服务端上，先查看redis缓存发现没有数据，则数据库压力增大。

解决方案：

- 布隆过滤器，限制请求，阻断在服务器外。
- 设置空值，先将数据返回，再及时更新redis，防止不一致情况。
- 数据预热。

![image-20210531165535710](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210531165535710.png)

将数据通过若干hash函数映射到一维数组空间，增加和查询时间复杂度为O(K)，k为hash函数个数，如果查询到有一个不为1，则一定不存在，如果全为1，则有可能存在，原因在于hash冲突。减少hash冲突的方法是增加hash函数个数和一维数组长度，这在redis集成的布隆过滤器中可通过参数配置。

#### 缓存击穿

针对极个别热点数据，突然某一个时刻该缓存过期，导致大量线程访问数据库。

解决方案：

- 根据监控延长热点数据TTL
- 设置多级缓存，变相增加缓存有效时间
- 分布式锁，限制单个key的访问只能有一个线程去查数据库，结果缓存后释放锁。

#### 缓存雪崩

大量热点数据同一时间失效，导致大量请求访问到数据库。

解决方案：

- 异地多活，部署集群，扩容、减负
- 限流降级，关闭其他不相关服务，同时针对热点数据访问进行降级，比如只有一个线程访问数据库，之后立即缓存。
- 设置随机过期时间，尽量让这些热点数据不在同一时刻过期。

### 分布式锁

