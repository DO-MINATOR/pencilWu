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

| **命令**   | **RDB**    | **AOF**      |
| ---------- | ---------- | ------------ |
| 启动优先级 | 低         | 高           |
| 体积       | 小         | 大           |
| 恢复速度   | 快         | 慢           |
| 数据安全性 | 容易丢数据 | 根据策略决定 |

#### 混和持久化

```
# aof-use-rdb-preamble yes   
```

在记录时，将之前的一部分数据进行RDB化，新的AOF命令追加到后面，这样混和持久化文件结构如下：

![image-20210929110642905](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210929110642905.png)

### 备份策略

1. crontab定时将rdb/aof文件写入到一个新目录中。
2. 超过30天备份自动清除。

### 主从架构

Master写，Slave读。通过读写分离，实现性能扩展，以及容灾的快速恢复（Salve因为有多个，因此读服务器可以自动完成切换，而Master只有一个，因此主机如果想要实现容灾备份的话，需要集群）

- 创建多个redis-6379.conf、redis-6380.conf、redis-6381.conf
  - include /redis/redis.conf，引入完整配置
  - pidfile /var/run/redis_6379.pid
  - port 6379
  - dbfilename dump6379.rdb
- 启动三服务，redis-server redis-6379.conf
- info replication查看服务器属性
- slaveof 127.0.0.1 6379，在6380、6381上配置从属信息

#### 工作原理

- 主动复制：当salve初次连接到master时，会进行主动的全量更新（slave挂掉重启后依然可以完成）
- 被动传输：在master每次更新数据后，将增量信息主动发送给slave

#### 薪火相传

![image-20210929111358840](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210929111358840.png)

中间的slave既是从机又是主机，后面的从机通过slaveof命令配置中间节点为其master，重新连接时，将进行新的全量更新。这样可以避免单点压力。

### Jedis

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

```java
public class JedisSingleTest {
    public static void main(String[] args) throws IOException {

        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(20);
        jedisPoolConfig.setMaxIdle(10);
        jedisPoolConfig.setMinIdle(5);

        // timeout，这里既是连接超时又是读写超时，从Jedis 2.8开始有区分connectionTimeout和soTimeout的构造函数
        JedisPool jedisPool = new JedisPool(jedisPoolConfig, "192.168.0.60", 6379, 3000, null);

        Jedis jedis = null;
        try {
            //从redis连接池里拿出一个连接执行命令
            jedis = jedisPool.getResource();

            System.out.println(jedis.set("single", "zhuge"));
            System.out.println(jedis.get("single"));

            //管道示例
            //管道的命令执行方式：cat redis.txt | redis-cli -h 127.0.0.1 -a password - p 6379 --pipe
            /*Pipeline pl = jedis.pipelined();
            for (int i = 0; i < 10; i++) {
                pl.incr("pipelineKey");
                pl.set("zhuge" + i, "zhuge");
            }
            List<Object> results = pl.syncAndReturnAll();
            System.out.println(results);*/

            //lua脚本模拟一个商品减库存的原子操作
            //lua脚本命令执行方式：redis-cli --eval /tmp/test.lua , 10
            /*jedis.set("product_count_10016", "15");  //初始化商品10016的库存
            String script = " local count = redis.call('get', KEYS[1]) " +
                            " local a = tonumber(count) " +
                            " local b = tonumber(ARGV[1]) " +
                            " if a >= b then " +
                            "   redis.call('set', KEYS[1], a-b) " +
                            "   return 1 " +
                            " end " +
                            " return 0 ";
            Object obj = jedis.eval(script, Arrays.asList("product_count_10016"), Arrays.asList("10"));
            System.out.println(obj);*/

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //注意这里不是关闭连接，在JedisPool模式下，Jedis会被归还给资源池。
            if (jedis != null)
                jedis.close();
        }
    }
}
```

#### Pipeline管道

客户端可以一次发送多条redis请求，而不是每发一次请求等待一次网络IO，降低了IO开销，管道内的命令相互隔离，执行成功与否不影响其他命令执行。

#### Lua脚本

同样可以减少IO开销。重要的是lua脚本将作为一个整体执行，具有原子性和事务性，redis内置了lua解释器，通过eval执行脚本命令。

`EVAL script numkeys key [key ...] arg [arg ...]`

如

```
127.0.0.1:6379> eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
```

### 哨兵模式

反客为主的自动化实现，通过配置哨兵，自动监测Master是否宕机，如果宕机，则按照一定规则选举一台Slave成为Master，之后其余Slave以及重启后的旧的Master将变为Slaves。

哨兵也是redis服务，但不提供读写功能，主要用来监控redis实例节点。

- 配置sentinel.conf，添加命令`sentinel monitor mymaster 127.0.0.1 6379 1`，mymaster为哨兵的名称，1为至少需要多少个哨兵同意的数量。
- 启动哨兵，`redis-sentinel ./sentinel.conf`

sentinel集群都启动完毕后，会将哨兵集群的元数据信息写入所有sentinel的配置文件里去(追加在文件的最下面)，我们查看下如下配置文件sentinel-26379.conf，如下所示：

```
sentinel known-replica mymaster 192.168.0.60 6380 #代表redis主节点的从节点信息
sentinel known-replica mymaster 192.168.0.60 6381 #代表redis主节点的从节点信息
sentinel known-sentinel mymaster 192.168.0.60 26380 52d0a5d70c1f90475b4fc03b6ce7c3c56935760f  #代表感知到的其它哨兵节点
sentinel known-sentinel mymaster 192.168.0.60 26381 e9f530d3882f8043f76ebb8e1686438ba8bd5ca6  #代表感知到的其它哨兵节点
```

当redis主节点如果挂了，哨兵集群会重新选举出新的redis主节点，同时会修改所有sentinel节点配置文件的集群元数据信息，比如6379的redis如果挂了，假设选举出的新主节点是6380，则sentinel文件里的集群元数据信息会变成如下所示：

```
sentinel known-replica mymaster 192.168.0.60 6379 #代表主节点的从节点信息
sentinel known-replica mymaster 192.168.0.60 6381 #代表主节点的从节点信息
sentinel known-sentinel mymaster 192.168.0.60 26380 52d0a5d70c1f90475b4fc03b6ce7c3c56935760f  #代表感知到的其它哨兵节点
sentinel known-sentinel mymaster 192.168.0.60 26381 e9f530d3882f8043f76ebb8e1686438ba8bd5ca6  #代表感知到的其它哨兵节点
```

同时还会修改sentinel文件里之前配置的mymaster对应的6379端口，改为6380

```
sentinel monitor mymaster 192.168.0.60 6380 2
```

哨兵监控master-salve节点情况，master挂掉后，自动切换slave为新的master，这样客户端可以继续申请redis服务。

```java
public class JedisSentinelTest {
    public static void main(String[] args) throws IOException {

        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(20);
        config.setMaxIdle(10);
        config.setMinIdle(5);

        String masterName = "mymaster";
        Set<String> sentinels = new HashSet<String>();
        sentinels.add(new HostAndPort("192.168.0.60",26379).toString());
        sentinels.add(new HostAndPort("192.168.0.60",26380).toString());
        sentinels.add(new HostAndPort("192.168.0.60",26381).toString());
        //JedisSentinelPool其实本质跟JedisPool类似，都是与redis主节点建立的连接池
        //JedisSentinelPool并不是说与sentinel建立的连接池，而是通过sentinel发现redis主节点并与其建立连接
        JedisSentinelPool jedisSentinelPool = new JedisSentinelPool(masterName, sentinels, config, 3000, null);
        Jedis jedis = null;
        try {
            jedis = jedisSentinelPool.getResource();
            System.out.println(jedis.set("sentinel", "zhuge"));
            System.out.println(jedis.get("sentinel"));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //注意这里不是关闭连接，在JedisPool模式下，Jedis会被归还给资源池。
            if (jedis != null)
                jedis.close();
        }
    }
}
```

通过sentinel（哨兵服务）获取jedis连接对象，这样Master挂掉后，又会重新连接上新的Master。

### RedisTemplate

```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.5.4</version>
</dependency>
```

RedisTemplate中定义了对5种数据结构操作     

```
redisTemplate.opsForValue();//操作字符串
redisTemplate.opsForHash();//操作hash
redisTemplate.opsForList();//操作list
redisTemplate.opsForSet();//操作set
redisTemplate.opsForZSet();//操作有序set
```

| **String类型结构**     |                                           |
| ---------------------- | ----------------------------------------- |
| Redis                  | RedisTemplate rt                          |
| set key value          | rt.opsForValue().set("key","value")       |
| get key                | rt.opsForValue().get("key")               |
| del key                | rt.delete("key")                          |
| strlen key             | rt.opsForValue().size("key")              |
| getset key value       | rt.opsForValue().getAndSet("key","value") |
| getrange key start end | rt.opsForValue().get("key",start,end)     |
| append key value       | rt.opsForValue().append("key","value")    |

| **Hash结构**                             |                                                       |
| ---------------------------------------- | ----------------------------------------------------- |
| hmset key field1 value1 field2 value2... | rt.opsForHash().putAll("key",map) //map是一个集合对象 |
| hset key field value                     | rt.opsForHash().put("key","field","value")            |
| hexists key field                        | rt.opsForHash().hasKey("key","field")                 |
| hgetall key                              | rt.opsForHash().entries("key") //返回Map对象          |
| hvals key                                | rt.opsForHash().values("key") //返回List对象          |
| hkeys key                                | rt.opsForHash().keys("key") //返回List对象            |
| hmget key field1 field2...               | rt.opsForHash().multiGet("key",keyList)               |
| hsetnx key field value                   | rt.opsForHash().putIfAbsent("key","field","value"     |
| hdel key field1 field2                   | rt.opsForHash().delete("key","field1","field2")       |
| hget key field                           | rt.opsForHash().get("key","field")                    |

| **List结构**                                               |                                                   |
| ---------------------------------------------------------- | ------------------------------------------------- |
| lpush list node1 node2 node3...                            | rt.opsForList().leftPush("list","node")           |
| rt.opsForList().leftPushAll("list",list) //list是集合对象  |                                                   |
| rpush list node1 node2 node3...                            | rt.opsForList().rightPush("list","node")          |
| rt.opsForList().rightPushAll("list",list) //list是集合对象 |                                                   |
| lindex key index                                           | rt.opsForList().index("list", index)              |
| llen key                                                   | rt.opsForList().size("key")                       |
| lpop key                                                   | rt.opsForList().leftPop("key")                    |
| rpop key                                                   | rt.opsForList().rightPop("key")                   |
| lpushx list node                                           | rt.opsForList().leftPushIfPresent("list","node")  |
| rpushx list node                                           | rt.opsForList().rightPushIfPresent("list","node") |
| lrange list start end                                      | rt.opsForList().range("list",start,end)           |
| lrem list count value                                      | rt.opsForList().remove("list",count,"value")      |
| lset key index value                                       | rt.opsForList().set("list",index,"value")         |

| **Set结构**                                        |                                                             |
| -------------------------------------------------- | ----------------------------------------------------------- |
| sadd key member1 member2...                        | rt.boundSetOps("key").add("member1","member2",...)          |
| rt.opsForSet().add("key", set) //set是一个集合对象 |                                                             |
| scard key                                          | rt.opsForSet().size("key")                                  |
| sidff key1 key2                                    | rt.opsForSet().difference("key1","key2") //返回一个集合对象 |
| sinter key1 key2                                   | rt.opsForSet().intersect("key1","key2")//同上               |
| sunion key1 key2                                   | rt.opsForSet().union("key1","key2")//同上                   |
| sdiffstore des key1 key2                           | rt.opsForSet().differenceAndStore("key1","key2","des")      |
| sinter des key1 key2                               | rt.opsForSet().intersectAndStore("key1","key2","des")       |
| sunionstore des key1 key2                          | rt.opsForSet().unionAndStore("key1","key2","des")           |
| sismember key member                               | rt.opsForSet().isMember("key","member")                     |
| smembers key                                       | rt.opsForSet().members("key")                               |
| spop key                                           | rt.opsForSet().pop("key")                                   |
| srandmember key count                              | rt.opsForSet().randomMember("key",count)                    |
| srem key member1 member2...                        | rt.opsForSet().remove("key","member1","member2",...)        |