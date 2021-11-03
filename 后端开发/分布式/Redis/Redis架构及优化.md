## 架构相关问题

### 架构设计

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211008195856209.png" alt="image-20211008195856209" style="zoom:67%;" />

### 缓存穿透

指查询一个根本不存在的数据， 缓存层和存储层都不会命中，黑客模拟大量此类请求，可以造成存储层访问压力过大。

解决方案

- 缓存空对象

```java
String get(String key) {
    // 从缓存中获取数据
    String cacheValue = cache.get(key);
    // 缓存为空
    if (StringUtils.isBlank(cacheValue)) {
        // 从存储中获取
        String storageValue = storage.get(key);
        cache.set(key, storageValue);
        // 如果存储数据为空， 需要设置一个过期时间(300秒)
        if (storageValue == null) {
            cache.expire(key, 60 * 5);
        }
        return storageValue;
    } else {
        // 缓存非空
        return cacheValue;
    }
}
```

- 布隆过滤器

对于恶意攻击，向服务器请求大量不存在的数据造成的缓存穿透，可以用布隆过滤器先做一次过滤，对于不存在的数据布隆过滤器一般都能够过滤掉，不让请求再往后端发送。**布隆过滤器**判断如果不存在，则一定不存在，如果存在，则有可能不存在。

![image-20211008200334748](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211008200334748.png)

原理是一个超大bit位数组，和无偏hash函数，通过将key经过几次hash后，设置某一位为1，这样可以大大降低hash碰撞的概率，查找某个key时，如果全部为1，则有可能存在，如果有一位为0，则一定不存在。相比设置空对象的方法，占用空间小很多。

```xml
<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson</artifactId>
   <version>3.6.5</version>
</dependency>
```

```java
public class RedissonBloomFilter {

    public static void main(String[] args) {
        Config config = new Config();
        config.useSingleServer().setAddress("redis://localhost:6379");
        //构造Redisson
        RedissonClient redisson = Redisson.create(config);

        RBloomFilter<String> bloomFilter = redisson.getBloomFilter("nameList");
        //初始化布隆过滤器：预计元素为100000000L,误差率为3%,根据这两个参数会计算出底层的bit数组大小
        bloomFilter.tryInit(100000000L,0.03);
        //将zhuge插入到布隆过滤器中
        bloomFilter.add("zhuge");

        //判断下面号码是否在布隆过滤器中
        System.out.println(bloomFilter.contains("guojia"));//false
        System.out.println(bloomFilter.contains("baiqi"));//false
        System.out.println(bloomFilter.contains("zhuge"));//true
    }
}
```

使用布隆过滤器需要把所有数据提前放入布隆过滤器，如果修改了原始数据，布隆过滤器只能重新初始化。

### 缓存击穿

由于大批量缓存在同一时间失效可能导致大量请求同时穿透缓存直达数据库，可能会造成数据库瞬间压力过大甚至挂掉，对于这种情况我们在批量增加缓存时最好将这一批数据的缓存过期时间设置为一个时间段内的不同时间。例如秒杀场景。

```java
String get(String key) {
    // 从缓存中获取数据
    String cacheValue = cache.get(key);
    // 缓存为空
    if (StringUtils.isBlank(cacheValue)) {
        // 从存储中获取
        String storageValue = storage.get(key);
        cache.set(key, storageValue);
        //设置一个过期时间(300到600之间的一个随机数)
        int expireTime = new Random().nextInt(300)  + 300;
        cache.expire(key, expireTime);
        return storageValue;
    } else {
        // 缓存非空
        return cacheValue;
    }
}
```

### 缓存雪崩

通常是由于缓存层遭到了超过支撑上限的请求，或是bigkey等设计原因，导致请求又打到了存储层，有可能导致级联宕机。

解决方案

1. 合理设计集群。
2. 保护缓存层，使用限流、熔断、降级等机制。
   比如访问一些非核心数据时，可以直接返回预定义的默认信息，降低后台查询复杂。如果访问核心数据时，才经过缓存层。
3. 使用异步请求队列，将请求放入队列中，依次处理。

### 热点缓存重建

如果热点key突然出现，例如大v推荐的某一个冷门商品，此时大量请求有可能同时达到存储层。

解决方案，通过分布式锁，限制只有一个线程可以去查询数据库，并重建缓存。

```java
String get(String key) {
    // 从Redis中获取数据
    String value = redis.get(key);
    // 如果value为空， 则开始重构缓存
    if (value == null) {
        // 只允许一个线程重建缓存， 使用nx， 并设置过期时间ex
        String mutexKey = "mutext:key:" + key;
        if (redis.set(mutexKey, "1", "ex 180", "nx")) {
             // 从数据源获取数据
            value = db.get(key);
            // 回写Redis， 并设置过期时间
            redis.setex(key, timeout, value);
            // 删除key_mutex
            redis.delete(mutexKey);
        }// 其他线程休息50毫秒后重试
        else {
            Thread.sleep(50);
            get(key);
        }
    }
    return value;
}
```

### 缓存和数据库双写不一致

1.双写不一致

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211008201947417.png" alt="image-20211008201947417" style="zoom:67%;" />

2.读写不一致

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211008202044524.png" alt="image-20211008202044524" style="zoom:67%;" />

- 对于个人的业务数据，如订单、购物车，几乎不会发生不一致问题，通常设置过期时间即可，定期主动更新。
- 并发量大的数据，但非核心业务，可以容忍一段时间的不一致，同样设置过期时间，定时主动更新。
- 使用读写锁可以很好的避免不一致问题，同时最小化该机制带来的性能下降。（读读的时候，不会发生不一致问题，有写线程时，排队执行，等待缓存更新完成）

注意，对于写多的场景，一般不考虑缓存层了，因为数据的强一致性，需要尽快写入到数据库。

## 开发优化

### 键值设计

避免bigkey，通常list、hash、set中容易出现该问题，(string也会，最大支持512MB)，二级数据结构可以存储2^32^-1个元素。实际场景中，如果满足一下两点，则认为bigkey：

1. string超过10KB
2. 二级结构存储元素过多，超过5000个。

**bigkey的危害：**

1. 操作bigkey比较花时间，可能会造成redis阻塞。
2. 网络拥塞，bigkey对应主机的网卡流量会很大。
3. 删除时，如果是lrem、srem或者hdel，有可能造成阻塞，因此通常采用渐进式删除，hscan，每次遍历一小段，直到下标返回0。

**bigkey的产生：**

一般来说，bigkey的产生都是由于程序设计不当，或者对于数据规模预料不清楚造成的，来看几个例子：

1. 社交类：粉丝列表，如果某些明星或者大v不精心设计下，必是bigkey。
2. 统计类：例如按天存储某项功能或者网站的用户集合，除非没几个人用，否则必是bigkey。
3. 缓存类：将数据从数据库load出来序列化放到Redis里，这个方式非常常用，但有两个地方需要注意，第一，是不是有必要把所有字段都缓存；第二，有没有相关关联的数据，有的同学为了图方便把相关数据都存一个key下，产生bigkey。

**优化bigkey**

1. 拆分
   big list： list1、list2、...listN
   big hash：可以讲数据分段存储，比如一个大的key，假设存了1百万的用户数据，可以拆分成200个key，每个key下面存放5000个用户数据

2. 如果bigkey不可避免，也要思考一下要不要每次把所有元素都取出来(例如有时候仅仅需要hmget，而不是hgetall)，删除也是一样，尽量使用优雅的方式来处理。

### 命令使用

- O(N)命令关注N的数量
  例如hgetall、lrange、smembers、zrange、sinter等并非不能使用，但是需要明确N的值。有遍历的需求可以使用hscan、sscan、zscan代替。

- 禁用命令
  禁止线上使用keys、flushall、flushdb等，通过redis的rename机制禁掉命令，或者使用scan的方式渐进式处理。

- redis应用隔离
  不同业务采用不同的redis集群，防止单线程带来不必要的性能下降。
- 批量操作减小IO比重
  如mget、mset等命令，pipeline，lua脚本（注意不能太复杂，否则也会阻塞）

### 连接池

```java
JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
jedisPoolConfig.setMaxTotal(5);
jedisPoolConfig.setMaxIdle(2);
jedisPoolConfig.setTestOnBorrow(true);

JedisPool jedisPool = new JedisPool(jedisPoolConfig, "192.168.0.60", 6379, 3000, null);

Jedis jedis = null;
try {
    jedis = jedisPool.getResource();
    //具体的命令
    jedis.executeCommand()
} catch (Exception e) {
    logger.error("op key {} error: " + e.getMessage(), key, e);
} finally {
    //注意这里不是关闭连接，在JedisPool模式下，Jedis会被归还给资源池。
    if (jedis != null) 
        jedis.close();
}
```

| 参数名   | 含义                       | 默认值 |
| -------- | -------------------------- | ------ |
| maxTotal | 资源池中最大连接数         | 8      |
| maxIdle  | 资源池允许最大空闲的连接数 | 8      |
| minIdle  | 资源池确保最少空闲的连接数 | 0      |

实际中，一般看maxIdle而不是minIdle，当访问量下来后，空闲连接数会保持在maxIdle。另外，设置maxTotal=maxIdle，可以避免增缩容。

如果系统已启动就需要立即访问，那么可以做一个redis连接池预热。

```java
List<Jedis> minIdleJedisList = new ArrayList<Jedis>(jedisPoolConfig.getMinIdle());

for (int i = 0; i < jedisPoolConfig.getMinIdle(); i++) {
    Jedis jedis = null;
    try {
        jedis = pool.getResource();
        minIdleJedisList.add(jedis);
        jedis.ping();
    } catch (Exception e) {
        logger.error(e.getMessage(), e);
    } finally {
        //注意，这里不能马上close将连接还回连接池，否则最后连接池里只会建立1个连接。。
        //jedis.close();
    }
}
//统一将预热的连接还回连接池
for (int i = 0; i < jedisPoolConfig.getMinIdle(); i++) {
    Jedis jedis = null;
    try {
        jedis = minIdleJedisList.get(i);
        //将连接归还回连接池
        jedis.close();
    } catch (Exception e) {
        logger.error(e.getMessage(), e);
    } finally {
    }
}
```

### 清理策略

1. 被动删除：碰到一个过期的key时才触发删除。
2. 主动删除：redis定期清理过期数据。
3. 如果超过预定义的maxmemory，则主动清理，机制如下：
   - 针对过期key
     1. volatile-ttl：在筛选时，会针对设置了过期时间的键值对，根据过期时间的先后进行删除，越早过期的越先被删除。
     2. volatile-random：就像它的名称一样，在设置了过期时间的键值对中，进行随机删除。
     3. volatile-lru：会使用 LRU 算法筛选设置了过期时间的键值对删除。
     4. volatile-lfu：会使用 LFU 算法筛选设置了过期时间的键值对删除。
   - 针对所有key
     1. allkeys-random：从所有键值对中随机选择并删除数据。
     2. allkeys-lru：使用 LRU 算法在所有数据中进行筛选删除。
     3. allkeys-lfu：使用 LFU 算法在所有数据中进行筛选删除。

lru，最近访问时间作参考，一般场景都适用，另一种是lfu，参考最近访问次数，适用于时间无关的集中访问。