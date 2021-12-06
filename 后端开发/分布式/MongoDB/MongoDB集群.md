### 复制集架构

提供高可用，数据写入主节点时，同步到副节点上。主节点发生故障，自动切换副节点。

- 数据分发：将数据从一个区域复制到另一个区域，减少另一个区域的读延迟
- 读写分离：不同类型的压力分别在不同的节点上执行
- 异地容灾：在数据中心故障时快速切换到异地

**典型复制集结构**

一个典型的复制集由三个或三个以上具有投票权的节点组成，其中一个主节点（Primary）：接收写入操作，读操作和选举时投票，两个或多个从节点(Secondary)：复制主节点上的新数据和选举时投票

**数据是如何复制的？**

当一个修改操作，无论是插入，更新或删除，到达主节点时，它对数据的操作将被记录下来（经过一些必要的转换）。这些记录称为oplog

从节点通过从主节点上不断获取新进入主节点的oplog，并在自己的数据上回放，以此保持跟主节点的数据一致。

![image-20211201173646179](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211201173646179.png)

**复制集注意事项**

硬件：

因为正常的复制集节点都有可能成为主节点，它们的地位是一样的，因此硬件配置上必须一致

为了保证节点不会同时宕机，各节点的硬件必须具有独立性。

软件：

复制集各节点软件版本必须一致，以避免出现不可预知的问题

增加节点不会增加系统写性能

### 集群配置

**创建数据目录文件**

Linux系统 

mkdir -p /data/db{1,2,3}

**准备每个数据库的配置文件**

复制集的每个mongod进程应该位于不同的服务器。我们现在在一台服务器上运行三个实例，所以要为它们各自配置

1.不同的端口： 28017， 28018，28019.

2.不同的数据目录

data/db1,data/db2,data/db3

**不同的日志文件路径**。

/data/db1/mongod.log

/data/db2/mongod.log

/data/db3/mongod.log

```conf
#data/db1/mongod.conf 
systemLog:
  destination: file
  path: /data/db1/mongod.log
  logAppend: true
storage:
  dbPath: /data/db1
net:
  bindIp: 0.0.0.0
  port: 28017
replication:
  replSetName: rs0
processManagement:
  fork: true
```

```conf
#/data/db2/mongod.conf 
systemLog:
  destination: file
  path: /data/db2/mongod.log
  logAppend: true
storage:
  dbPath: /data/db2
net:
  bindIp: 0.0.0.0
  port: 28018
replication:
  replSetName: rs0
processManagement:
  fork: true
```

```conf
#/data/db3/mongod.conf 
systemLog:
  destination: file
  path: /data/db3/mongod.log
  logAppend: true
storage:
  dbPath: /data/db3
net:
  bindIp: 0.0.0.0
  port: 28019
replication:
  replSetName: rs0
processManagement:
  fork: true
```

启动三个实例

mongod -f /data/db1/mongod.conf

mongod -f /data/db2/mongod.conf

mongod -f /data/db3/mongod.conf

配置主节点

```js
 mongo --port 28017
rs.initiate({
_id:"rs0",
members:[{
_id:0,
host:"localhost:28017"
},{
_id:1,
host:"localhost:28018"
},{
_id:2,
host:"localhost:28019"
}]
})
```

mongo --port 28017 连接到主节点

![image-20211201175227232](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211201175227232.png)

mongo --port 28018  连接到从节点上面

![image-20211201175243467](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211201175243467.png)

### 集群分片

数据量突破单机瓶颈，数据量大，恢复很慢，不利于数据管理。并发量突破单机性能瓶颈。

- 应用全透明
- 数据自动均衡
- 动态扩容，无需下线

![image-20211201175657514](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211201175657514.png)

**分片集群角色：**

- 路由节点： mongos， 提供集群单一入口，转发应用端请求，选择合适的数据节点进行读写，合并多个数据节点的返回。无状态，建议  mongos节点集群部署以提供高可用性。客户请求应发给mongos，而不是 分片服务器，当查询包含分片片键时，mongos将查询发送到指定分片，否则，mongos将查询发送到所有分片，并汇总所有查询结果。 

- 配置节点: ：就是普通的mongod进程， 建议以复制集部署，提供高可用。提供集群元数据存储分片数据分布的数据。主节点故障时，配置服务器进入只读模式。只读模式下，数据段分裂和集群平衡都不可执行。整个复制集故障时，分片集群不可用 

- 数据节点：以复制集为单位，横向扩展最大1024分片，分片之间数据不重复，所有数据在一起才可以完整工作。

**分片键：**

- 范围分片

比如 key  的值 从 min -  max

可以把数据进行范围分片

- hash分片

通过 hash(key ) 进行数据分段

片键值用来将集合中的文档划分为数据段,片键必须对应一个索引或索引前缀（单键、复合键）,可以使用片键的值 或者片键值的哈希值进行分片

**选择片键**

- 片键值的范围更广（可以使用复合片键扩大范围）
- 片键值的分布更平衡（可使用复合片键平衡分布）
- 片键值不要单向增大、减小（可使用哈希片键）  

**数据段的分裂**

当数据段尺寸过大，或者包含过多文档时，触发数据段分裂,只有新增、更新文档时才可能自动触发数据段分裂,数据段分裂通过更新元数据来实现。

**集群的平衡**

后台运行的平衡器负责监视和调整集群的平衡,当最大和最小分片之间的数据段数量相差过大时触发，集群中添加或移除分片时也会触发。