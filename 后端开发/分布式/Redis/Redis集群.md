### 集群

之前的哨兵模式存在访问瞬断问题，即在选举新的master时服务会中断数秒。相较集群，只有一个cluster提供服务，并发量不大。

解决生产环境中的容量、写操作压力分担。通过无中心化集群，在连接集群服务时，可自动将请求打在不同服务器上。

![image-20210929154307597](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210929154307597.png)

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

### 槽位Slot

一个 Redis 集群包含 16384 个插槽（hash slot）， 数据库中的每个键都属于这 16384 个插槽的其中一个， 集群使用公式 CRC16(key) % 16384 来计算键 key 属于哪个槽。每个slot可能发生“hash”冲突。

```txt
cluster keyslot cust //返回key cust所在的slot号
cluster countkeysinslot 4847 //返回4847号slot中拥有的keys的数量
cluster getkeysinslot 4847 10 //返回4847号slot中10个key
```

**注意：**上述命令需要slot段对应的节点上(Master/Slave)执行才有效

当客户端发送了一个命令，服务端发现该key对应的hash值并不在自己管理的slot段中，会发送一个重定向命令给客户端，同时更新客户端的slot缓存表。这样下次客户端发送求命令可直接发送到正确的服务器上。

### 集群通信机制

采用gossip协议进行通信，维护集群的主从角色，节点数量，槽位信息。

- meet：节点发送meet给新加入的节点，让其进入到集群中，之后新节点开始和其他节点进行通信。
- ping：向其他节点发送的心跳信息，包含自身元数据信息。
- pong：响应ping请求。
- fail：移除节点通知信息。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/116144643385.gif" alt="116144643385" style="zoom:67%;" />

gossip协议的优势在于分散了节点维护的压力，但只保证了最终一致性。会出现不一致问题。gossip协议通信端口为服务端口+10000。

### 故障恢复与新master选举

如果Master挂掉后，对应的Slave会自动晋升为Master，原来的Master恢复后，自动变为Slave。

1. slave发现自己的master变为FAIL
2. 将自己记录的集群currentEpoch加1，并广播FAILOVER_AUTH_REQUEST 信息
3. 其他节点收到该信息，只有master响应，判断请求者的合法性，并发送FAILOVER_AUTH_ACK，对每一个epoch只发送一次ack
4. 尝试failover的slave收集master返回的FAILOVER_AUTH_ACK
5. slave收到超过半数master的ack后变成新Master(这里解释了集群为什么至少需要三主个节点，如果只有两个，当其中一个挂了，只剩一个主节点是不能选举成功的)
6. slave广播Pong消息通知其他集群节点。

如果某一段slot的Master和Slave都挂掉后，会根据cluster-require-full-coverage对应的配置是否为yes决定集群服务是否还能运行，如果还能运行，只是当前数据段无法工作。

### 脑裂问题

如果因为网络抖动导致一个cluster下出现多个master，则客户端会将写数据请求到两个不同的节点中，当网络恢复后，会产生数据不一致，最终成为slave的旧的master会丢失部分数据。为此可在redis.conf中添加

```conf
min-replicas-to-write 1 //写数据成功最少同步的slave数量，这样出现网络问题的那台master就会出现写不成功。避免了丢失数据。
```

### 对于批量命令的支持

对于像mset这样需要请求不同节点的命令，由于mset是原子的，但不同槽位的可用性不同，因此只支持key在同一个cluster的mset命令。

### Jedis集群

```java
public class JedisClusterTest {
    public static void main(String[] args) throws IOException {

        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(20);
        config.setMaxIdle(10);
        config.setMinIdle(5);

        Set<HostAndPort> jedisClusterNode = new HashSet<HostAndPort>();
        jedisClusterNode.add(new HostAndPort("192.168.0.61", 8001));
        jedisClusterNode.add(new HostAndPort("192.168.0.62", 8002));
        jedisClusterNode.add(new HostAndPort("192.168.0.63", 8003));
        jedisClusterNode.add(new HostAndPort("192.168.0.61", 8004));
        jedisClusterNode.add(new HostAndPort("192.168.0.62", 8005));
        jedisClusterNode.add(new HostAndPort("192.168.0.63", 8006));

        JedisCluster jedisCluster = null;
        try {
            //connectionTimeout：指的是连接一个url的连接等待时间
            //soTimeout：指的是连接上一个url，获取response的返回等待时间
            jedisCluster = new JedisCluster(jedisClusterNode, 6000, 5000, 10, "zhuge", config);
            //"zhuge"为密码
            System.out.println(jedisCluster.set("cluster", "zhuge"));
            System.out.println(jedisCluster.get("cluster"));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (jedisCluster != null)
                jedisCluster.close();
        }
    }
}

运行效果如下：
OK
zhuge
```

