### 架构图

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/sdadsd.jpg" alt="sdadsd" style="zoom: 33%;" />

#### 核心控制器controller

在Kafka集群中会有一个或者多个broker，其中有一个broker会被选举为控制器（Kafka Controller），它负责管理整个集群中所有分区和副本的状态。

- 当某个分区的leader副本出现故障时，由控制器负责为该分区选举新的leader副本。
- 当检测到某个分区的ISR集合发生变化时，由控制器负责通知所有broker更新其元数据信息。
- 当使用kafka-topics.sh脚本为某个topic增加分区数量时，同样还是由控制器负责让新分区被其他节点感知到。

#### leader选举机制

在kafka集群启动的时候，会自动选举一台broker作为controller来管理整个集群，选举的过程是集群中每个broker都会尝试在zookeeper上创建一个 **/controller 临时节点**，zookeeper会保证有且仅有一个broker能创建成功，这个broker就会成为集群的总控器controller。

#### leader作用

- 监听broker相关的变化。为Zookeeper中的/brokers/ids/节点添加BrokerChangeListener，用来处理broker增减的变化。
- 监听topic相关的变化。为Zookeeper中的/brokers/topics节点添加TopicChangeListener，用来处理topic增减的变化；为Zookeeper中的/admin/delete_topics节点添加TopicDeletionListener，用来处理删除topic的动作。
- 从Zookeeper中读取获取当前所有与topic、partition以及broker有关的信息并进行相应的管理。对于所有topic所对应的Zookeeper中的/brokers/topics/[topic]节点添加PartitionModificationsListener，用来监听topic中的分区分配变化。
- 当某个partition的leader挂了后，通过controller选举出该partition对应的新的leader，一般取isr列表中的第一个节点作为leader。
- 更新集群的元数据信息，同步到其他普通的broker节点中。

### 消费者记录offset的机制

每个consumer会定期将自己消费分区的offset提交给kafka内部topic：**__consumer_offsets**，提交过去的时候，key是**consumerGroupId+topic+分区号**，value就是当前offset的值，kafka会定期清理topic里的消息，最后就保留最新的那条数据

 因为__consumer_offsets可能会接收高并发的请求，kafka默认给其**分配50个分区**(可以通过offsets.topic.num.partitions设置)，这样可以通过加机器的方式抗大并发。

Kafka的log_data目录下，除了有分区目录，如partition_1、partition_2，还有这个记录offset的文件。每次消费者重新拉取消息时，都可以通过该文件获取到具体offset。

### 消费者Rebalance机制

rebalance就是说如果消费组里的消费者数量有变化或消费的分区数有变化，kafka会重新分配消费者消费分区的关系。比如consumer group中某个消费者挂了，此时会自动把分配给他的分区交给其他的消费者，如果他又重启了，那么又会把一些分区重新交还给他。

**注意：**rebalance只针对subscribe这种不指定分区消费的情况，如果通过assign这种消费方式指定了分区，kafka不会进行rebanlance。

如下情况可能会触发消费者rebalance

1. 消费组里的consumer增加或减少了
2. 动态给topic增加了分区
3. 消费组订阅了更多的topic

rebalance过程中，消费者无法从kafka消费消息，这对kafka的TPS会有影响，如果kafka集群内节点较多，比如数百个，那重平衡可能会耗时极多，所以应尽量避免在系统高峰期的重平衡发生。

### 消息发布机制

#### 1、写入方式

producer 采用 push 模式将消息发布到 broker，每条消息都被 append 到 patition 中，属于顺序写磁盘（顺序写磁盘效率比随机写内存要高，保障 kafka 吞吐率）。

#### 2、消息路由

producer 发送消息到 broker 时，会根据分区算法选择将其存储到哪一个 partition。其路由机制为：

1. 指定了 patition，则直接使用。
2. 未指定 patition 但指定 key，通过对 key 的 value 进行hash 选出一个 patition。
3. patition 和 key 都未指定，使用轮询选出一个 patition。

#### 3、写入流程

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/d.png" alt="d" style="zoom: 50%;" />

1. producer 先从 zookeeper 的 "/brokers/.../state" 节点找到该 partition 的 leader
2. producer 将消息发送给该 leader
3. leader 将消息写入本地 log
4. followers 从 leader pull 消息，写入本地 log 后 向leader 发送 ACK
5. leader 收到**所有 ISR 中**的 replica 的 ACK 后，增加 HW（high watermark，最后 commit 的 offset） 并向 producer 发送 ACK

### 日志分段存储

Kafka 一个分区的消息数据对应存储在一个文件夹下，以topic名称+分区号命名，消息在分区内是分段(segment)存储，每个段的消息都存储在不一样的log文件里，这种特性方便old segment file快速删除，kafka规定了一个段位的 log 文件最大为 1G，做这个限制目的是为了方便把 log 文件加载到内存去操作：

```bash
# 部分消息的offset索引文件，kafka每次往分区发4K(可配置)消息就会记录一条当前消息的offset到index文件，
# 如果要定位消息的offset会先在这个文件里快速定位，再去log文件里找具体消息
00000000000000000000.index
# 消息存储文件，主要存offset和消息体
00000000000000000000.log
# 消息的发送时间索引文件，kafka每次往分区发4K(可配置)消息就会记录一条当前消息的发送时间戳与对应的offset到timeindex文件，
# 如果需要按照时间来定位消息的offset，会先在这个文件里查找
00000000000000000000.timeindex

00000000000005367851.index
00000000000005367851.log
00000000000005367851.timeindex
```

