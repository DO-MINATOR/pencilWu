### 简介

Kafka是最初由Linkedin公司开发，是一个分布式、支持分区的（partition）、多副本的（replica），基于zookeeper协调的分布式消息系统，它的最大的特性就是可以实时的处理大量数据以满足各种需求场景：比如基于hadoop的批处理系统、低延迟的实时系统、Storm/Spark流式处理引擎，web/nginx日志、访问日志，消息服务等等，用scala语言编写，Linkedin于2010年贡献给了Apache基金会并成为顶级开源 项目。

### 使用场景

- 日志收集：一个公司可以用Kafka收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、Hbase、Solr等。
- 消息系统：解耦和生产者和消费者、缓存消息等。
- 用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘。
- 运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/kafka日志平台架构.png" alt="kafka日志平台架构" style="zoom: 50%;" />

### 相关概念

| **名称**      | **解释**                                                     |
| ------------- | ------------------------------------------------------------ |
| Broker        | 消息中间件处理节点，一个Kafka节点就是一个broker，一个或者多个Broker可以组成一个Kafka集群 |
| Topic         | Kafka根据topic对消息进行归类，发布到Kafka集群的每条消息都需要指定一个topic |
| Producer      | 消息生产者，向Broker发送消息的客户端                         |
| Consumer      | 消息消费者，从Broker读取消息的客户端                         |
| ConsumerGroup | 一个topic下的partition只能被ConsumerGroup下的一个consumer消费 |
| Partition     | 物理上的概念，一个topic可以分为多个partition，每个partition内部消息是有序的 |

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/Kafka集群 (1).jpg" alt="Kafka集群 (1)" style="zoom: 33%;" />

#### Topic

![image-20211108114854632](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211108114854632.png)

为什么要对Topic下数据进行分区存储？

1. commit log文件会受到所在机器的文件系统大小的限制，分区之后可以将不同的分区放在不同的机器上，相当于对数据做了**分布式存储**，理论上一个topic可以处理任意数量的数据。
2. 为了**提高并行度**。分区里每条消息都有唯一的offset，消费组也记录偏移量，用于决定从哪接受消息。

#### Producers

生产者将消息发送到topic中去，同时负责选择将message发送到topic的哪一个partition中。通过round-robin做简单的负载均衡。也可以根据消息中的某一个关键字来进行区分。通常第二种方式使用的更多。

#### Consumers

传统的消息传递模式有2种：队列( queue) 和（publish-subscribe）

- queue模式：多个consumer从服务器中读取数据，消息只会到达一个consumer。
- publish-subscribe模式：消息会被广播给所有的consumer。

Kafka基于这2种模式提供了一种consumer的抽象概念：consumer group。

- queue模式：所有的consumer都位于同一个consumer group 下。
- publish-subscribe模式：所有的consumer都有着自己唯一的consumer group。

![image-20211108120537674](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211108120537674.png)

#### 消费顺序

一个partition同一个时刻在一个consumer group中只能有一个consumer instance在消费，从而保证消费顺序。

**consumer group中的consumer instance的数量不能比一个Topic中的partition的数量多，否则，多出来的consumer消费不到消息。**

Kafka只在partition的范围内保证消息消费的局部顺序性，不能在同一个topic中的多个partition中保证总的消费顺序性。

如果有在总体上保证消费顺序的需求，那么我们可以通过将topic的partition数量设置为1，将consumer group中的consumer instance数量也设置为1，但是这样会影响性能，所以kafka的顺序消费很少用。

### 基本使用

kafka依赖zookeeper，所以需要先安装zookeeper

```bash
wget https://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.5.8/apache-zookeeper-3.5.8-bin.tar.gz
tar -zxvf apache-zookeeper-3.5.8-bin.tar.gz
cd  apache-zookeeper-3.5.8-bin
cp conf/zoo_sample.cfg conf/zoo.cfg

# 启动zookeeper
bin/zkServer.sh start
bin/zkCli.sh 
ls /			#查看zk的根目录相关节点
```

下载Kafka

```bash
wget https://mirror.bit.edu.cn/apache/kafka/2.4.1/kafka_2.11-2.4.1.tgz  # 2.11是scala的版本，2.4.1是kafka的版本
tar -xzf kafka_2.11-2.4.1.tgz
cd kafka_2.11-2.4.1
```

修改config/server.properties

```bash
#broker.id属性在kafka集群中必须要是唯一
broker.id=0
#kafka部署的机器ip和提供服务的端口号
listeners=PLAINTEXT://192.168.65.60:9092   
#kafka的消息存储文件
log.dir=/usr/local/data/kafka-logs
#kafka连接zookeeper的地址
zookeeper.connect=192.168.65.60:2181
```

启动Kafka

```bash
# 启动kafka，运行日志在logs目录的server.log文件里
bin/kafka-server-start.sh -daemon config/server.properties   #后台启动，不会打印日志到控制台
或者用
bin/kafka-server-start.sh config/server.properties &
# 我们进入zookeeper目录通过zookeeper客户端查看下zookeeper的目录树
bin/zkCli.sh 
ls /		#查看zk的根目录kafka相关节点
ls /brokers/ids	#查看kafka节点
# 停止kafka
bin/kafka-server-stop.sh
```

**相关配置参数**

| **Property**               | **Default**                    | **Description**                                              |
| -------------------------- | ------------------------------ | ------------------------------------------------------------ |
| broker.id                  | 0                              | 每个broker都可以用一个唯一的非负整数id进行标识；这个id可以作为broker的“名字”，你可以选择任意你喜欢的数字作为id，只要id是唯一的即可。 |
| log.dirs                   | /tmp/kafka-logs                | kafka存放数据的路径。这个路径并不是唯一的，可以是多个，路径之间只需要使用逗号分隔即可；每当创建新partition时，都会选择在包含最少partitions的路径下进行。 |
| listeners                  | PLAINTEXT://192.168.65.60:9092 | server接受客户端连接的端口，ip配置kafka本机ip即可            |
| zookeeper.connect          | localhost:2181                 | zooKeeper连接字符串的格式为：hostname:port，此处hostname和port分别是ZooKeeper集群中某个节点的host和port；zookeeper如果是集群，连接方式为 hostname1:port1, hostname2:port2, hostname3:port3 |
| log.retention.hours        | 168                            | 每个日志文件删除之前保存的时间。默认数据保存时间对所有topic都一样。 |
| num.partitions             | 1                              | 创建topic的默认分区数                                        |
| default.replication.factor | 1                              | 自动创建topic的默认副本数量，建议设置为大于等于2             |
| min.insync.replicas        | 1                              | 当producer设置acks为-1时，min.insync.replicas指定replicas的最小数目（必须确认每一个repica的写数据都是成功的），如果这个数目没有达到，producer发送消息会产生异常 |
| delete.topic.enable        | false                          | 是否允许删除主题                                             |

创建主题

```bash
bin/kafka-topics.sh --create --zookeeper 192.168.65.60:2181 --replication-factor 1 --partitions 1 --topic test
#命令来查看kafka中目前存在的topic
bin/kafka-topics.sh --list --zookeeper 192.168.65.60:2181
#创建多分区主题
bin/kafka-topics.sh --create --zookeeper 192.168.65.60:2181 --replication-factor 1 --partitions 2 --topic test1
#查看下topic的情况
bin/kafka-topics.sh --describe --zookeeper 192.168.65.60:2181 --topic test1
```

![dsa](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/dsa.png)

删除主题

```bash
bin/kafka-topics.sh --delete --topic test --zookeeper 192.168.65.60:2181
```

发送消息

```bash
bin/kafka-console-producer.sh --broker-list 192.168.65.60:9092 --topic test 
>this is a msg
```

接收消息

```bash
bin/kafka-console-consumer.sh --bootstrap-server 192.168.65.60:9092 --topic test   
#如果想要消费之前的消息可以通过--from-beginning参数指定，如下命令：
bin/kafka-console-consumer.sh --bootstrap-server 192.168.65.60:9092 --from-beginning --topic test 
#消费多主题
bin/kafka-console-consumer.sh --bootstrap-server 192.168.65.60:9092 --whitelist "test|test-2"
#消费组里只有个一消费者可以接收消息
bin/kafka-console-consumer.sh --bootstrap-server 192.168.65.60:9092  --consumer-property group.id=testGroup --topic test
#不同消费组可以消费相同topic
bin/kafka-console-consumer.sh --bootstrap-server 192.168.65.60:9092 --consumer-property group.id=testGroup-2 --topic test 
#查看消费组名
bin/kafka-consumer-groups.sh --bootstrap-server 192.168.65.60:9092 --list 
#查看消费组的消费偏移量
bin/kafka-consumer-groups.sh --bootstrap-server 192.168.65.60:9092 --describe --group testGroup
```

![clipboard](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/clipboard.png)

- **current-offset：**当前消费组的已消费偏移量
- **log-end-offset：**主题对应分区消息的结束偏移量(HW)
- **lag：**当前消费组未消费的消息数

### 集群实战

伪集群配置

```bash
cp config/server.properties config/server-1.properties
cp config/server.properties config/server-2.properties
```

```bash
broker.id=1
listeners=PLAINTEXT://192.168.65.60:9093   
log.dir=/usr/local/data/kafka-logs-1
zookeeper.connect=192.168.65.60:2181

broker.id=2
listeners=PLAINTEXT://192.168.65.60:9094
log.dir=/usr/local/data/kafka-logs-2
zookeeper.connect=192.168.65.60:2181
```

启动两个实例

```bash
bin/kafka-server-start.sh -daemon config/server-1.properties
bin/kafka-server-start.sh -daemon config/server-2.properties
```

创建多副本多分区主题

```bash
bin/kafka-topics.sh --create --zookeeper 192.168.65.60:2181 --replication-factor 3 --partitions 2 --topic my-replicated-topic
```

发送消息

```bash
bin/kafka-console-producer.sh --broker-list 192.168.65.60:9092,192.168.65.60:9093,192.168.65.60:9094 --topic my-replicated-topic
>my test msg 1
```

消费消息

```bash
bin/kafka-console-consumer.sh --bootstrap-server 192.168.65.60:9092,192.168.65.60:9093,192.168.65.60:9094 --from-beginning --topic my-replicated-topic
my test msg 1
```

kill掉一个broker

```bash
ps -ef | grep server.properties
kill 14776
```

查看主题

```bash
bin/kafka-topics.sh --describe --zookeeper 192.168.65.60:2181 --topic my-replicated-topic
```

![adasds](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/adasds.png)

kafka将很多集群关键信息记录在zookeeper里，保证自己的无状态，从而在水平扩容时非常方便。

log的partitions分布在kafka集群中不同的broker上，每个broker可以请求备份其他broker上partition上的数据。kafka集群支持配置一个partition备份的数量。

针对每个partition，都有一个broker起到“leader”的作用，0个或多个其他的broker作为“follwers”的作用。**leader处理所有的针对这个partition的读写请求，而followers被动复制leader的结果，不提供读写(主要是为了保证多副本数据与消费的一致性)**。如果这个leader失效了，其中的一个follower将会自动的变成新的leader。

### java测试

```xml
<dependency>
   <groupId>org.apache.kafka</groupId>
   <artifactId>kafka-clients</artifactId>
   <version>2.4.1</version>
</dependency>
```

```java
//生产者
public class MsgProducer {
    private final static String TOPIC_NAME = "my-replicated-topic";

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.65.60:9092,192.168.65.60:9093,192.168.65.60:9094");
         /*
         发出消息持久化机制参数
        （1）acks=0： 表示producer不需要等待任何broker确认收到消息的回复，就可以继续发送下一条消息。性能最高，但是最容易丢消息。
        （2）acks=1： 至少要等待leader已经成功将数据写入本地log，但是不需要等待所有follower是否成功写入。就可以继续发送下一
             条消息。这种情况下，如果follower没有成功备份数据，而此时leader又挂掉，则消息会丢失。
        （3）acks=-1或all： 需要等待 min.insync.replicas(默认为1，推荐配置大于等于2) 这个参数配置的副本个数都成功写入日志，这种策略会保证
            只要有一个备份存活就不会丢失数据。这是最强的数据保证。一般除非是金融级别，或跟钱打交道的场景才会使用这种配置。
         */
        /*props.put(ProducerConfig.ACKS_CONFIG, "1");
         *//*
        发送失败会重试，默认重试间隔100ms，重试能保证消息发送的可靠性，但是也可能造成消息重复发送，比如网络抖动，所以需要在
        接收者那边做好消息接收的幂等性处理
        *//*
        props.put(ProducerConfig.RETRIES_CONFIG, 3);
        //重试间隔设置
        props.put(ProducerConfig.RETRY_BACKOFF_MS_CONFIG, 300);
        //设置发送消息的本地缓冲区，如果设置了该缓冲区，消息会先发送到本地缓冲区，可以提高消息发送性能，默认值是33554432，即32MB
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
        *//*
        kafka本地线程会从缓冲区取数据，批量发送到broker，
        设置批量发送消息的大小，默认值是16384，即16kb，就是说一个batch满了16kb就发送出去
        *//*
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        *//*
        默认值是0，意思就是消息必须立即被发送，但这样会影响性能
        一般设置10毫秒左右，就是说这个消息发送完后会进入本地的一个batch，如果10毫秒内，这个batch满了16kb就会随batch一起被发送出去
        如果10毫秒内，batch没满，那么也必须把消息发送出去，不能让消息的发送延迟时间太长
        *//*
        props.put(ProducerConfig.LINGER_MS_CONFIG, 10);*/
        //把发送的key从字符串序列化为字节数组
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        //把发送消息value从字符串序列化为字节数组
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        Producer<String, String> producer = new KafkaProducer<String, String>(props);

        int msgNum = 5;
        final CountDownLatch countDownLatch = new CountDownLatch(msgNum);
        for (int i = 1; i <= msgNum; i++) {
            Order order = new Order(i, 100 + i, 1, 1000.00);
            //指定发送分区
            /*ProducerRecord<String, String> producerRecord = new ProducerRecord<String, String>(TOPIC_NAME
                    , 0, order.getOrderId().toString(), JSON.toJSONString(order));*/
            //未指定发送分区，具体发送的分区计算公式：hash(key)%partitionNum
            ProducerRecord<String, String> producerRecord = new ProducerRecord<String, String>(TOPIC_NAME
                    , order.getOrderId().toString(), JSON.toJSONString(order));
            //order.getOrderId().toString()作为key

            //等待消息发送成功的同步阻塞方法
            /*RecordMetadata metadata = producer.send(producerRecord).get();
            System.out.println("同步方式发送消息结果：" + "topic-" + metadata.topic() + "|partition-"
                    + metadata.partition() + "|offset-" + metadata.offset());*/

            //异步回调方式发送消息
            producer.send(producerRecord, new Callback() {
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    if (exception != null) {
                        System.err.println("发送消息失败：" + exception.getStackTrace());

                    }
                    if (metadata != null) {
                        System.out.println("异步方式发送消息结果：" + "topic-" + metadata.topic() + "|partition-"
                                + metadata.partition() + "|offset-" + metadata.offset());
                    }
                    countDownLatch.countDown();
                }
            });
            //TODO
        }
        countDownLatch.await(5, TimeUnit.SECONDS);
        producer.close();
    }
}
```

```java
//消费者
public class MsgConsumer {
    private final static String TOPIC_NAME = "my-replicated-topic";
    private final static String CONSUMER_GROUP_NAME = "testGroup";

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.65.60:9092,192.168.65.60:9093,192.168.65.60:9094");
        // 消费分组名
        props.put(ConsumerConfig.GROUP_ID_CONFIG, CONSUMER_GROUP_NAME);
        // 是否自动提交offset，默认就是true
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        // 自动提交offset的间隔时间
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
        //props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
        /*
        当消费主题的是一个新的消费组，或者指定offset的消费方式，offset不存在，那么应该如何消费
        latest(默认) ：只消费自己启动之后发送到主题的消息
        earliest：第一次从头开始消费，以后按照消费offset记录继续消费，这个需要区别于consumer.seekToBeginning(每次都从头开始消费)
        */
        //props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
      /*
      consumer给broker发送心跳的间隔时间，broker接收到心跳如果此时有rebalance发生会通过心跳响应将
      rebalance方案下发给consumer，这个时间可以稍微短一点
      */
        props.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 1000);
        /*
        服务端broker多久感知不到一个consumer心跳就认为他故障了，会将其踢出消费组，
        对应的Partition也会被重新分配给其他consumer，默认是10秒
        */
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 10 * 1000);
        //一次poll最大拉取消息的条数，如果消费者处理速度很快，可以设置大点，如果处理速度一般，可以设置小点
        props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 500);
        /*
        如果两次poll操作间隔超过了这个时间，broker就会认为这个consumer处理能力太弱，
        会将其踢出消费组，将分区分配给别的consumer消费
        */
        props.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 30 * 1000);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);

        consumer.subscribe(Arrays.asList(TOPIC_NAME));
        // 消费指定分区
        //consumer.assign(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));

        //消息回溯消费
        //consumer.seekToBeginning(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));

        //指定offset消费
        /*consumer.assign(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));
        consumer.seek(new TopicPartition(TOPIC_NAME, 0), 10);*/

        //从指定时间点开始消费
        /*List<PartitionInfo> topicPartitions = consumer.partitionsFor(TOPIC_NAME);
        //从1小时前开始消费
        long fetchDataTime = new Date().getTime() - 1000 * 60 * 60;
        Map<TopicPartition, Long> map = new HashMap<>();
        for (PartitionInfo par : topicPartitions) {
            map.put(new TopicPartition(topicName, par.partition()), fetchDataTime);
        }
        Map<TopicPartition, OffsetAndTimestamp> parMap = consumer.offsetsForTimes(map);
        for (Map.Entry<TopicPartition, OffsetAndTimestamp> entry : parMap.entrySet()) {
            TopicPartition key = entry.getKey();
            OffsetAndTimestamp value = entry.getValue();
            if (key == null || value == null) continue;
            Long offset = value.offset();
            System.out.println("partition-" + key.partition() + "|offset-" + offset);
            System.out.println();
            //根据消费里的timestamp确定offset
            if (value != null) {
                consumer.assign(Arrays.asList(key));
                consumer.seek(key, offset);
            }
        }*/
        while (true) {
            /*
             * poll() API 是拉取消息的长轮询
             */
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("收到消息：partition = %d,offset = %d, key = %s, value = %s%n", record.partition(),
                        record.offset(), record.key(), record.value());
            }
            /*if (records.count() > 0) {
                // 手动同步提交offset，当前线程会阻塞直到offset提交成功
                // 一般使用同步提交，因为提交之后一般也没有什么逻辑代码了
                consumer.commitSync();

                // 手动异步提交offset，当前线程提交offset不会阻塞，可以继续处理后面的程序逻辑
                consumer.commitAsync(new OffsetCommitCallback() {
                    @Override
                    public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
                        if (exception != null) {
                            System.err.println("Commit failed for " + offsets);
                            System.err.println("Commit failed exception: " + exception.getStackTrace());
                        }
                    }
                });
            }*/
        }
    }
}
```

