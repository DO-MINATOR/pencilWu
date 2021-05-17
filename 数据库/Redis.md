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

### 基础命令

- 切换数据库：`select id`
- 查看当前数据库key数量：`dbsize`
- 清空当前库：`flushdb`
- 清空全部库：`flushall`
- `keys *`查看当前库所有key  (匹配：keys *1)
- `exists key`判断某个key是否存在
- `type key` 查看你的key是什么类型
- `del key`    删除指定的key数据
- `unlink key`  根据value选择非阻塞删除
- `expire key 10`  10秒钟：为给定的key设置过期时间
- `ttl key` 查看还有多少秒过期，-1表示永不过期，-2表示已过期

### String

最基础的数据类型，可以存放二进制数据，此时value大小不能超过512M。

set  \<key>\<value>添加键值对

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

数据结构为简单动态字符串，类似于ArrayList，采用动态扩容方式。

### List

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

### Set

value是set类型，相比list，可以自动去重，底层实际上是value为null的hash表，时间复杂度为O(1)。

- sadd \<key>\<value1>\<value2> ..... 
- 将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略
- smembers \<key>取出该集合的所有值。
- sismember \<key>\<value>判断集合\<key>是否为含有该\<value>值，有1，没有0
- scard\<key>返回该集合的元素个数。
- srem \<key>\<value1>\<value2> .... 删除集合中的某个元素。
- spop \<key>**随机从该集合中吐出一个值。**
- srandmember \<key>\<n>随机从该集合中取出n个值。不会从集合中删除 。
- smove \<source>\<destination>value把集合中一个值从一个集合移动到另一个集合
- sinter \<key1>\<key2>返回两个集合的交集元素。
- sunion \<key1>\<key2>返回两个集合的并集元素。
- sdiff \<key1>\<key2>返回两个集合的**差集**元素(key1中的，不包含key2中的)

### Hash

value是hashmap类型，适用于存储非结构化对象。同样，当存储数量较少时，使用ziplist（压缩列表），map对象多时，改为hashtable。

- hset \<key>\<field>\<value>给\<key>集合中的 \<field>键赋值\<value>
- hget \<key1>\<field>从\<key1>集合\<field>取出 value 
- hmset \<key1>\<field1>\<value1>\<field2>\<value2>... 批量设置hash的值
- hexists\<key1>\<field>查看哈希表 key 中，给定域 field 是否存在。 
- hkeys \<key>列出该hash集合的所有field
- hvals \<key>列出该hash集合的所有value
- hincrby \<key>\<field>\<increment>为哈希表 key 中的域 field 的值加上增量 1  -1
- hsetnx \<key>\<field>\<value>将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 

### Zset

