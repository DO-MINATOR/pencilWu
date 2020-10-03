1、Spark Streaming

低延时、容错性、可扩展性、可以集合图计算、机器学习完成多功能实时流计算。

2、工作原理

  大体：按照数据接受的时间片划分为batch交予Spark core进行“离线处理”

![image-20200520175602299](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520175602299.png)

  执行流程：集群中1台节点接受inputstream，并划分batch块，将块拷贝至集群其他节点，同时反馈块信息至NameNode，节点内部执行streaming流处理。

![image-20200520175606210](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520175606210.png)

3、核心组件

StreamingContext类似于SparkSession，用于读取指定source下的流数据。

Dstream类似与Spark SQL中的RDD，只不过它是流式的。

4、streaming提供了两大类输入流，一种是file system、socket connection，另一种是Kafka、Flume等

5、使用local模式，注意线程数一定要>receivers，因为接受数据使用单独的线程（除了file system的数据）

6、spark streaming编程实战：

  引入spark-streaming_2.11依赖

  **编写基于socket的wordcount：**

```scala
val sparkConf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
val ssc = new StreamingContext(sparkConf, Seconds(5))
//创建streaming的context，配置切片大小
val lines = ssc.socketTextStream("192.168.1.233", 6789)
val result = lines.flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _)
result.print()
//设置数据源，中间操作，输出结果
ssc.start()//开始过程
ssc.awaitTermination()//等待结束
```

  **编写基于filesystem的wordcount：**

```scala
val sparkConf = new SparkConf().setMaster("local").setAppName("FileWordCount")
val sc = new StreamingContext(sparkConf, Seconds(5))
val lines = sc.textFileStream("hdfs://hadoop000:8020/")
val result = lines.flatMap(_.split(" ")).map(x => (x, 1)).reduceByKey(_ + _)
result.print()
sc.start()
sc.awaitTermination();
```

需要注意的是，指定的只能是单极的目录，且只能通过生成（移动、创建）文件的方式到该目录下。

  **编写基于updateStateByKey带状态算子的wordcount：**

  带状态，即把之前的统计结果写入到当前批次里的数据。需要注意ssc须设置checkpoint目录（用于保存历史数据）

```scala
def updateFunction(cur: Seq[Int], pre: Option[Int]): Option[Int] = {
  val curnum = cur.sum
  val prenum = pre.getOrElse(0)
  Some(curnum + prenum)
}//编写状态更新函数
val result = lines.flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _)
val state = result.updateStateByKey[Int](updateFunction _)//调用
state.print()
```

  **将结果输出到数据库**

```scala
result.foreachRDD(rdd => {
  rdd.foreachPartition(partionOfRecords => {
    val connection = ConnectionPool.getConnction()
    partionOfRecords.foreach(record => {
      val sql = "insert into wordcount values('" + record._1 + "'," + "'" + record._2 + "')"
      connection.createStatement().execute(sql)
    })
    ConnectionPool.free(connection)
  })
})
```

  注意foreachRDD实在driver上执行，foreachPartition和foreach实在work上执行，而connection不支持序列化并传输，因此会报错。

  **编写基于window的wordcount:**

```scala
val windowDuration = Seconds(10)
val slideDuration = Seconds(5)
val lines = ssc.socketTextStream("192.168.1.233", 6789)
val result = lines.flatMap(_.split(" ")).map((_, 1)).reduceByKeyAndWindow((a:Int, b:Int) => a + b, windowDuration, slideDuration)
```

  注意，window和slide需要为时间片的整数倍。

  **编写transform实现高级规则:**

```scala
val lines = ssc.socketTextStream("192.168.1.233", 6789)
lines.map(x=>(x.split(",")(1),x)).transform(rdd=>{rdd.leftOuterJoin(blacksRDD).filter(x=>x._2._2.getOrElse(false)!=true).map(_._2._1)}).print()
```

  transform用于将Dstream转换为RDD进行操作，并返回Dstream。

  **编写基于spark sql的wordcount：**

```scala
words.foreachRDD {
  (rdd: RDD[String], time: Time) =>
    val spark = SparkSessionSingleton.getInstance(rdd.sparkContext.getConf)
    import spark.implicits._
    val wordsDataFrame = rdd.map(w => Record(w)).toDF()
    wordsDataFrame.createOrReplaceTempView("words")

    val wordCountsDataFrame =
      spark.sql("select word, count(*) as total from words group by word")
    println(s"========= $time =========")
    wordCountsDataFrame.show()
}
```

  **Spark Streaming** **整合Flume开发(Push)：**

```scala
val flumeStream = FlumeUtils.createStream(ssc, "192.168.1.5",41414)
flumeStream.map(x=> new String(x.event.getBody.array()).trim)
  .flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).print()
```

  注意，先启动接收器，因为flume启动的是一个推流

  **Spark Streaming** **整合Flume开发(Poll)：**

```scala
val flumeStream = FlumeUtils.createPollingStream(ssc, "192.168.1.5",41414)
flumeStream.map(x=> new String(x.event.getBody.array()).trim)
  .flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).print()
```

  此时，应先启动flume，相较于第一种方式，poll拉流更加可靠。

  **spark Streaming** **整合Kafka开发（receiver）：**

```scala
val Array(zkQuorum, group, topics, numThreads) = args
val sparkConf = new SparkConf()
val ssc = new StreamingContext(sparkConf, Seconds(3))

val topicMap = topics.split(",").map((_, numThreads.toInt)).toMap
val messages = KafkaUtils.createStream(ssc, zkQuorum, group, topicMap)
messages.map(x => x._2).flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _).print()
```

  结合Kafka需要zookeeper地址、group、topic队列名、numthreads参数。

  **spark Streaming** **整合Kafka开发（Direct）：**

```scala
val topicsSet = topics.split(",").toSet
val kafkaParams = Map[String, String]("metadata.broker.list" -> brokers)
val messages = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](ssc, kafkaParams, topicsSet)
```

  此时可以直接使用local[1]