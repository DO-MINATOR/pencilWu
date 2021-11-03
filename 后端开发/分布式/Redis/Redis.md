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

### 单线程模型

是指网络IO和键值对的读写都是一个线程来执行，即redis服务的请求都是需要排队实现的。但其他功能，比如主从同步等都是启动新进程帮助完成的。

正是因为所有操作都是在内存中进行的，且都是基于单线程模型，避免了多线程的切换，所以运算速度快，但也要注意避免某一个redis请求过于繁杂，导致整个redis服务卡顿。

### IO多路复用

NIO+epoll方式实现IO多路复用。

![image-20210929104546202](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210929104546202.png)

### 其他高级命令

`keys`：全表（正则）遍历，当redis数据量较大时，避免使用。

![image-20210929104753710](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210929104753710.png)

`scan`：渐进式遍历	SCAN cursor [MATCH pattern] [COUNT count]

第一个是开始遍历的下标值，下次遍历设值为当次遍历完成后的返回值，pattern正则，count为大概需要扫描的key的数量。结果不一定准确，因为新增或者hash扩容。

`info`：查看redis服务器信息。





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

可通过lua脚本，创建出一个事务环境，lua脚本执行具有原子性。

### 乐观/悲观锁

- 悲观锁：每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。
- 乐观锁：每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号机制(版本号的修改是序列化的)。乐观锁适用于多读的应用类型，这样可以提高吞吐量。Redis就是利用这种check-and-set机制实现事务的。

在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

秒杀系统中利用redis可能存在的问题及解决方案：

- 连接超时，使用连接池
- 超卖问题，使用乐观锁
- 乐观锁导致的库存遗留，通过循环或者lua脚本

### 分布式锁

针对秒杀系统中，大量请求针对同一个库存进行-1操作需要用到分布式锁机制。

如果是在单机系统中，通过JVM内置的synchronized锁即可保证操作的原子性，但在集群环境下，普通的锁无法保证操作的原子性，有可能有多个请求同时在不同服务器上执行库存-1操作，因此需要分布式锁提供保证。

已知的分布式锁的实现，有redis和zookeeper，前者性能最高，但在redis集群架构下会出现主从不一致情况，可能会出现bug，后者安全性最高，但性能略低。

```java
@RequestMapping("/deduct_stock")
public String deduckStock() {
    String lock_key = "product_001";
    String clientId = UUID.randomUUID().toString();
    RedisTemplate<String, Object> redisTemplate = null;
    try {
        redisTemplate = redisTemp.getRedisTemplate();
        Boolean result = redisTemplate.opsForValue().setIfAbsent(lock_key, clientId, 10, TimeUnit.SECONDS);
        if (!result) {
            return "error";
        }
        int stock = Integer.parseInt((String) redisTemplate.opsForValue().get("stock"));
        if (stock > 0) {
            int realStock = stock - 1;
            redisTemplate.opsForValue().set("stock", realStock + "");
            System.out.println("扣除成功，剩余库存：" + realStock + "");
        } else {
            System.out.println("库存不足");
        }
    } finally {
        if (clientId.equals(redisTemplate.opsForValue().get(lock_key))) {
            redisTemplate.delete(lock_key);
        }
    }
    return "succ";
}
```

这是一段接受客户端请求减库存的代码，其中用到了分布式锁。其原理大致如下：

![image-20210601170806214](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210601170806214.png)

由于redis的key只有一个，因此通过setnx命令可以上一个简单的排他锁，此时其他请求会阻塞在获取该锁的地方。如果只是简单的排他锁，还会出现一些bug：

> 如果业务中间出现错误或者服务器宕机，会出现永远无法释放锁的情况。

通过设置finally强制释放锁，以及设置锁过期时间，注意使用原子操作设置value和expire。

> 如果业务执行过程中，锁自动释放导致其他请求进来，违背了原子性。

为每一个进来的请求新开一个线程，定期检查该锁的存货情况，如果业务执行完毕前，该锁还存活，则延长其过期时间。

> 如果释放掉了其他请求的锁，也会导致锁失效。

设置锁的value为请求初始产生的UUID，并在释放时检查value是否一致，否则不释放。

`redisson`已经集成了上述功能。

另外，之前谈到的不一致问题在于Redis如果是主从架构的话，如果在Master收到锁命令后还没来得及向从服务器发送写命令就导致宕机切换的话，会导致这次上锁失败。可以考虑zookeeper一类拥有更强一致性的分布式框架。



