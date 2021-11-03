### Zookeeper客户端

客户端与服务端包含在一个jar包。

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.5.8</version>
</dependency>
```

**创建连接**


```java
@Slf4j
public class ZookeeperClientTest {

    private static final String ZK_ADDRESS="192.168.109.200:2181";
    private static final int SESSION_TIMEOUT = 5000;
    private static ZooKeeper zooKeeper;
    private static final String ZK_NODE="/zk-node";

    @Before
    public void init() throws IOException, InterruptedException {
        final CountDownLatch countDownLatch=new CountDownLatch(1);
        zooKeeper=new ZooKeeper(ZK_ADDRESS, SESSION_TIMEOUT, event -> {
            if (event.getState()== Watcher.Event.KeeperState.SyncConnected &&
                    event.getType()== Watcher.Event.EventType.None){
                countDownLatch.countDown();
                log.info("连接成功！");
            }
        });
        log.info("连接中....");
        countDownLatch.await();
    }
}
```

**同步创建节点**

```java
@Test
public void createTest() throws KeeperException, InterruptedException {
    String path = zooKeeper.create(ZK_NODE, "data".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    log.info("created path: {}",path);
}
```

**异步创建节点**

```java
@Test
public void createAsycTest() throws InterruptedException {
     zooKeeper.create(ZK_NODE, "data".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,
             CreateMode.PERSISTENT,
             (rc, path, ctx, name) -> log.info("rc  {},path {},ctx {},name {}",rc,path,ctx,name),"context");
    TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
}
```

**同步修改节点**

```java
@Test
public void setTest() throws KeeperException, InterruptedException {
    Stat stat = new Stat();
    byte[] data = zooKeeper.getData(ZK_NODE, false, stat);
    log.info("修改前: {}",new String(data));
    zooKeeper.setData(ZK_NODE, "changed!".getBytes(), stat.getVersion());
    byte[] dataAfter = zooKeeper.getData(ZK_NODE, false, stat);
    log.info("修改后: {}",new String(dataAfter));
}
```

### Curator客户端

Curator 是一套由netflix 公司开源的，Java 语言编程的 ZooKeeper 客户端框架，Curator项目是现在ZooKeeper 客户端中使用最多，对ZooKeeper 版本支持最好的第三方客户端，并推荐使用，Curator 把我们平时常用的很多 ZooKeeper 服务开发功能做了封装，例如 Leader 选举、分布式计数器、分布式锁。这就减少了技术人员在使用 ZooKeeper 时的大部分底层细节开发工作。在会话重新连接、Watch 反复注册、多种异常处理等使用场景中，用原生的 ZooKeeper 处理比较复杂。而在使用 Curator 时，由于其对这些功能都做了高度的封装，使用起来更加简单，不但减少了开发时间，而且增强了程序的可靠性。

两个 Curator 相关的包，第一个是 curator-framework 包，该包是对 ZooKeeper 底层 API 的一些封装。另一个是 curator-recipes 包，该包封装了一些 ZooKeeper 服务的高级特性，如：Cache 事件监听、选举、分布式锁、分布式 Barrier。

**创建连接**

```java
RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString("192.168.128.129:2181")
                .sessionTimeoutMs(5000)  // 会话超时时间
                .connectionTimeoutMs(5000) // 连接超时时间
                .retryPolicy(retryPolicy)
                .namespace("base") // 包含隔离名称
                .build();
client.start();
```

**创建节点**

```java
@Test
public void testCreate() throws Exception {
    String path = curatorFramework.create().forPath("/curator-node");
    // curatorFramework.create().withMode(CreateMode.PERSISTENT).forPath("/curator-node","some-data".getBytes())
    log.info("curator create node :{}  successfully.",path);
}
```

也可以一次性递归创建目录，原生客户端不支持

```java
@Test
public void testCreateWithParent() throws Exception {
    String pathWithParent="/node-parent/sub-node-1";
    String path = curatorFramework.create().creatingParentsIfNeeded().forPath(pathWithParent);
    log.info("curator create node :{}  successfully.",path);
}
```

**获取节点**

```java
@Test
public void testGetData() throws Exception {
    byte[] bytes = curatorFramework.getData().forPath("/curator-node");
    log.info("get data from  node :{}  successfully.",new String(bytes));
}
```

**异步获取节点**

```java
@Test
public void test() throws Exception {
    curatorFramework.getData().inBackground((client, event) -> {
        log.info(" background: {}", event);
    }).forPath(ZK_NODE);
    TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
}


@Test
public void test() throws Exception {
    ExecutorService executorService = Executors.newSingleThreadExecutor();
    curatorFramework.getData().inBackground((item1, item2) -> {
        log.info(" background: {}", item2);
    },executorService).forPath(ZK_NODE);
    TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
}//指定线程池执行异步方法
```

**更新节点**

```java
@Test
public void testSetData() throws Exception {
    curatorFramework.setData().forPath("/curator-node","changed!".getBytes());
    byte[] bytes = curatorFramework.getData().forPath("/curator-node");
    log.info("get data from  node /curator-node :{}  successfully.",new String(bytes));
}
```

**删除节点**

```java
@Test
public void testDelete() throws Exception {
    String pathWithParent="/node-parent";
    curatorFramework.delete().guaranteed().deletingChildrenIfNeeded().forPath(pathWithParent);
}
```

### Zookeeper集群

- Leader:   处理所有的事务请求（写请求），可以处理读请求，集群中只能有一个Leader
- Follower：只能处理读请求，同时作为 Leader的候选节点，即如果Leader宕机，Follower节点要参与到新的Leader选举中，有可能成为新的Leader节点。
- Observer：只能处理读请求。不能参与选举 

![image-20211026100338202](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211026100338202.png)

**下载解压zookeeper**

```sh
wget https://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.5.8/apache-zookeeper-3.5.8-bin.tar.gz
tar -zxvf apache-zookeeper-3.5.8-bin.tar.gz
cd  apache-zookeeper-3.5.8-bin
```

**重命名配置文件**

```sh
cp conf/zoo_sample.cfg conf/zoo-1.cfg
```

**修改每台机器的配置文件**

```sh
# vim conf/zoo-1.cfg
dataDir=/usr/local/data/zookeeper-1
clientPort=2181
server.1=127.0.0.1:2001:3001:participant// participant 可以不用写，默认就是participant
server.2=127.0.0.1:2002:3002:participant
server.3=127.0.0.1:2003:3003:participant
server.4=127.0.0.1:2004:3004:observer
```

- dataDir：顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。
- clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
- server.A=B：C：D：E 其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。如果需要添加不参与集群选举以及事务请求的过半机制的 Observer节点，可以在E的位置，添加observer标识。

**创建标识id**

```sh
 cd /usr/local/data/zookeeper-1
 vim myid
 1 
 cd /usr/local/data/zookeeper-2
 vim myid
 2 
 cd /usr/local/data/zookeeper-3
 vim myid
 3 
cd /usr/local/data/zookeeper-4
vim myid
4
```

**启动四个实例**

```sh
bin/zkServer.sh start conf/zoo1.cfg
bin/zkServer.sh start conf/zoo2.cfg
bin/zkServer.sh start conf/zoo3.cfg
bin/zkServer.sh start conf/coo4.cfg
```

**检测集群状态**

```sh
zkServer.sh   status conf/zoo1.cfg
```

![image-20211026101225865](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211026101225865.png)

**集群连接**

```sh
bin/zkCli.sh -server ip1:port1,ip2:port2,ip3:port3 
```

