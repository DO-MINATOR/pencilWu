### 简介

分布式协调框架，Hadoop的一个子应用，主要解决分布式应用程序中数据管理问题，如统一命名服务，状态同步服务，集群管理，分布式应用配置。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211011092435908.png" alt="image-20211011092435908" style="zoom:67%;" />

同样也是基于内存数据库，包含两个核心概念，文件数据结构+监听通知机制。

### 文件数据结构

类似于文件系统的数据结构。

![image-20211011092550084](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211011092550084.png)

支持对**目录**的增删改查和对**目录项中数据**的增删改查。

四种类型的Node。

1. Persistent 持久化节点
   客户端与zookeeper断开连接后，该节点依旧存在
2. Persistent_Sequential 持久化顺序节点
   客户端与zookeeper断开连接后，该节点依旧存在，且编号递增
3. Ephemeral 临时节点
   客户端与zookeeper断开连接后，该节点被删除
4. Ephemeral_Sequential 临时顺序节点
   客户端与zookeeper断开连接后，该节点被删除，且编号递增
5. Container 容器节点
   如果该节点下没有子节点，则自动删除

### 监听通知机制

1. 监听节点，则节点变化，或对应数据项变化，通知客户端。
2. 监听节点目录，则当节点结构发生变化时，通知客户端。
3. 监听节点递归目录，则当节点下的子节点结构变化时，通知客户端。

通知是一次性的，发生通知后，移除监听机制。

### 应用场景

- 分布式配置中心
- 分布式注册中心
- 分布式锁
- 分布式队列
- 集群选举
- 分布式屏障
- 发布/订阅

### 启动Zookeeper

**下载解压 zookeeper**

```
wget https://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.5.8/apache-zookeeper-3.5.8-bin.tar.gz
tar -zxvf apache-zookeeper-3.5.8-bin.tar.gz cd 
apache-zookeeper-3.5.8-bin     
```

**重命名配置文件  zoo_sample.cfg**   

```
cp zoo_sample.cfg  zoo.cfg
```

**启动zookeeper**

```
# 可以通过 bin/zkServer.sh  来查看都支持哪些参数  bin/zkServer.sh start conf/zoo.cfg
```

**连接zookeeper**

```
bin/zkCli.sh -server ip:port
```

**命令**

```
#创建节点，注意节点一定是以绝对路径开头
create [-e/-s/-c] path [data] [acl]
-e: 临时节点
-s: 顺序节点
-c: 容器节点
没有则默认持久化节点
data为携带的数据

#获取节点数据
get  /test-node

#修改节点数据
set /test-node some-data-changed

#查看节点状态
stat /test-node

#查看节点目录
ls /

#递归查看节点目录
ls -R /

#获取数据同时监听节点
get  -w  /path
#获取结构同时监听结构
ls -w /path
#获取结构同时监听递归结构
ls -w -R /path
```

### ACL权限控制

控制对节点的访问权限。包含3部分，**权限模式**（Scheme）、**授权对象**（ID）、**权限信息**（Permission）。最终组成一条例如`scheme:id:permission`格式的 ACL 请求信息。

scheme：

- 网段限制，比如我们可以让一个 IP 地址为“ip：192.168.0.110”的机器对服务器上的某个数据节点具有写入的权限。或者也可以通过“ip:192.168.0.1/24”给一段 IP 地址的机器赋权。
- 口令验证，客户端传送“username:password”这种形式的权限表示符。

id：

- 如果是网段限制，则对应具体ip或ip段
- 如果是口令验证，则对应一个用户名。

permission，操作类型限制：

- 数据节点（c: create）创建权限，授予权限的对象可以在数据节点下创建子节点；
- 数据节点（w: wirte）更新权限，授予权限的对象可以更新该数据节点；
- 数据节点（r: read）读取权限，授予权限的对象可以读取该节点的内容以及子节点的列表信息；
- 数据节点（d: delete）删除权限，授予权限的对象可以删除该数据节点的子节点；
- 数据节点（a: admin）管理者权限，授予权限的对象可以对该数据节点体进行 ACL 权限设置。

```
getAcl：获取某个节点的acl权限信息
setAcl：设置某个节点的acl权限信息
addauth: 注册用户信息

addauth digest gj:test
create /zk-node datatest digest:gj:X/NSthOB0fD/OT6iilJ55WJVado=:cdrwa 
#"gj:X/NSthOB0fD/OT6iilJ55WJVado="为test对应的sha-1密文

setAcl /node-ip ip:192.168.109.128:cdwra
create /node-ip  data  ip:192.168.109.128:cdwra
```

### 内存持久化

本身是一个基于内存的数据库，所以同样也有定期持久化和反序列化。

包含事务日志和数据快照。

- 事务日志：针对每一次客户端的事务操作，Zookeeper都会将他们记录到事务日志中，当然，Zookeeper也会将数据变更应用到内存数据库中。我们可以在zookeeper的主配置文件zoo.cfg 中配置内存中的数据持久化目录
- 数据快照：数据快照用于记录Zookeeper服务器上某一时刻的全量数据，并将其写入到指定的磁盘文件中。

日志文件是每次事务请求都会进行追加的操作，而快照是某时刻下的内存数据。恢复数据的时候，可以先恢复快照数据，再通过增量恢复事务日志中的数据即可。

