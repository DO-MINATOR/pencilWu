1、Kafka作为消息中间件（消息队列），可以缓冲生产者与消费者中间的数据流，具有可扩展、容错的特点。

2、架构

![image-20200520174630823](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520174630823.png)

生产者、消费者、broker（kafka消息队列）、topic（给消息流打上标签，用于区分不同数据流）

3、kafka环境配置

  首先需要下载zookeeper-cdh，并配置其下的zoo.cfg，通过zKServer.sh启动zookeeper进程。

  下载kafka-scala2.11，配置server.properties，其中：

broker.id

listeners

host.name

log.dirs

zookeeper.connect

  通过kafka-server-start.sh $KAFKA_HOME/config/server.properties启动kafka进程。

4、创建、查看队列，创建producer、consumer

`kafka-topics.sh --create --zookeeper hadoop000:2181 --replication-factor 1 --partitions 1 --topic hello_topic`

`kafka-topics.sh --list --zookeeper hadoop000:2181`

`kafka-topics.sh --describe --zookeeper hadoop000:2181 （--topic ...)查看详情`

`kafka-console-producer.sh --broker-list hadoop000:9092 --topic hello_topic`

`kafka-console-consumer.sh --zookeeper hadoop000:2181 --topic hello_topic（--from-beginning读取全部数据）`

使用

5、单机器多broker，需要后台启动多个Kafka-server-start.sh，如果只有&，则关闭终端，后台进程也会关闭，只有添加-daemon才会以后台方式运行。

kafka-topics.sh --create --zookeeper hadoop000:2181 --replication-factor 3 --partitions 1 --topic rep-topic（factor为3表示将之前启动的3个kafkaserver全部占用，以提升容错性）

`kafka-console-producer.sh --broker-list hadoop000:9093,hadoop000:9094,hadoop000:9095 --topic rep-topic`（启动生产者部署于3个broker之上）

`kafka-console-consumer.sh --zookeeper hadoop000:2181 --topic rep-topic`（启动1个消费者从zookeeper读取数据

6、Flume整合Kafka

![image-20200520175303737](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520175303737.png)

如图，Flume服务器将数据从log中输出到Kafka sink，Kafka将数据从消息队列中读取出来，输出到console。

首先修改配置文件avro-memory-kafka.conf：

![image-20200520175312199](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520175312199.png)

然后kafka-console-consumer.sh --zookeeper hadoop000:2181 --topic hello-topic启动Kafka消费者线程。