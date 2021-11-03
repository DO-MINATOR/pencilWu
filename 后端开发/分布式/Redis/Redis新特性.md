### Redis基本特性

1. 非关系型数据库，可以以O(1)时间复杂度操作数据库。
2. 键的类型不唯一，兼容客户端的所有类型，唯一。
3. 值可以是string、list、hash、set、sorted set。
4. redis内置了复制、持久化、lua脚本、事务、ssl、acl、客户端缓存等功能。
5. 通过集群机制提升性能。

### 应用场景

- 计数器
  对string进行增减操作，内存性能高，适合频繁读写。

- 分布式ID
  利用自增特性，一次生成步长稍大的id。

- 海量数据统计
  位图bitmap，如日活统计。

- 会话缓存
  类似于单点登录，可以将用户状态保存在redis中，这样应用服务器就脱离了状态，提升了扩展性和伸缩性。

- 分布式阻塞队列
  由于天生支持阻塞和单线程模型，可以实现分布式下的阻塞队列。

- 分布式锁
  多态应用服务器无法监控同一把本地锁，但可以借助redis的setnx命令实现分布式锁。

- 热点数据存储
  list

- 社交类关系数据

  set、list等

- 排行榜
  sorted set

### List

是一个有序队列（按添加数据排列），采用双端链表和ziplist进行存储，通过设置ziplist的单个大小，设置数据压缩范围，避免胖指针，提升数据存储效率。

```conf
list-max-ziplist-size  -2        //  单个ziplist节点最大能存储  8kb  ,超过则进行分裂,将数据存储在新的ziplist节点中
list-compress-depth  1        //  0 代表所有节点，都不进行压缩，1， 代表从头节点往后走一个，尾节点往前走一个不用压缩，其他的全部压缩，2，3，4 ... 以此类推
```

### Hash

底层数据结构采用hashtable，当数据量小或者单个元素小时，采用ziplist存储。

```conf
hash-max-ziplist-entries  512    //  ziplist 元素个数超过 512 ，将改为hashtable编码 
hash-max-ziplist-value    64      //  单个元素大小超过 64 byte时，将改为hashtable编码
```

### Sorted Set

skiplist实现，如果每层按两个节点数量提取的话，那么每往上提取一层，节点个数减半，最终需查找的次数为O(logn)。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211008152057308.png" alt="image-20211008152057308" style="zoom:67%;" />

### 新特性

#### 多线程IO

以前的redis执行流程为

`IO read->parse->resp->command execute->IO write`，即单个线程实现从网络IO到执行的全过程。

新版本使用多个线程协助执行网络IO操作，但仍只有main线程执行command execute，保留了原子性操作。

![image-20211008152614126](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211008152614126.png)

多线程IO默认是不开启的，需要再配置文件中配置

```config
io-threads-do-reads yes 
io-threads 4
```

#### 客户端缓存

客户端第一次拿到数据后，缓存，当服务器该key-value发生改变后，将主动通知客户端。

目前只有lettuce支持该特性。

#### ACL权限

创建用户，设置密码，设置可以操作的命令，具备某些前缀的key。

`acl list`查看当前用户权限

![image-20210605092930210](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210605092930210.png)

`acl setuser user1`创建user1

![image-20210605093039749](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210605093039749.png)

`acl setuser user2 on >password ~cached:* +get`创建并启用user2，设置密码，赋予get权限（需要key中含有cached字符串）

![image-20210605093228202](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210605093228202.png)