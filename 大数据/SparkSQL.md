HDFS优缺点：

  低容错、大数据、批处理

  高延迟、不适合小文件存储（块大小128MB）

Hive优点（类似HQL的语句）：

  适用于离线数据处理、底层支持不同执行引擎（MR、Spark、TEz...），与SparkSQL、Impala等使用统一的元数据管理

MapReduce缺点：

  使用不灵活且繁琐（只用Map和Reduce两个算子）、基于进程模型且频繁与IO打交道，导致效率低下

1、Spark简介，架设于HDFS之上，类似于MR离线处理，具有：

  高速处理，基于线程池模型，基于内存的列式存储

  编程模型简单易用

  以Spark为基础，可以搭建Spark SQL、Spark Streaming、MLlib等

![image-20200520173253092](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520173253092.png)

2、spark standalone模式，类似于HDFS、YARN的主从架构模式，可以脱离HADOOP运行，在conf/slaves中配置从节点，在conf/env.xml配置每个节点的core、memory等参数。然后通过sbin/star-all.sh启动master和worker，通过bin/spark-shell --master spark://hadoop000:7077连接主节点，并启动shell交互式进程。

3、spark local模式，模拟集群架构，使用方式和standalone模式相同，可以用于学习。

4、为何需要Hive/Spark SQL：

  a、使用QL语句对结构化数据进行统计分析比MR/Spark scala操作方便。

  b、以前的SQL关系型数据库无法存放大数据。

5、SparkSQL，广义上能够处理sql、hive、json、BI等外部结构化数据源，因此比HIVE更强大。且执行引擎和计划引擎基于原生Spark（线程模型，列式存储），因此效率更高。

6、SparkSQL架构

![image-20200520173439600](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520173314431.png)

前端可以使用多种结构化的外部数据源，而中间的逻辑执行引擎、计划引擎都采用原生Spark器件，效率会高很多。

7、Spark1.X版本中，有SQLContext、HIVEContext等多个上下文管理器，而2.X以后，统一封装为SparkSession。

8、编写SQLContext程序，打包jar，并运行在linux spark-submit --master local模式

spark-submit --class com.wsp.HiveContextApp --master local[2] --jars /home/hadoop/lib/mysql-connector-java-5.1.27-bin.jar /home/hadoop/lib/sparksql-1.0.jar

9、Spark只是一个计算框架，不具备存储环境，因此如果用到了Hadoop分布式环境下的文件，诸如hdfs://hadoop000/...或者hive数据表，则前提是hdfs环境已启动。

10、spark-shell中可以通过spark.sql进行与hive相关的操作，且执行效率比hive高，另外，可直接通过spark-sql接入sql相关的shell。(这一步需要拷贝hive/conf下的hive-site.xml文件)

11、通过start-thriftserver.sh --master local[2] --jars /home/hadoop/lib/mysql-connector-java-5.1.27-bin.jar启动一个thriftserver sparkapplication，然后通过beeline -u jdbc:hive2://hadoop000:10000 -n hadoop启动thrift线程。相比于spark-shell/sql的单进程方式，所有的beeline都是存在与一个thriftserver中，资源占用小，且能够实现表数据共享功能。

12、和spark-submit类似，thrift也有除beeline的shell交互式访问方式的API编程方式，需要引入hive-jdbc，然后使用类似jdbc的编程式访问hive数据。

13、Dataframe和RDD对比

都是分布式结构化数据集，但RDD基于行式存储，且无schema，Dataframe基于列式存储，内部有schema信息，因此，dataframe效率更高，API更加友好

![image-20200601095529038](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200601095529038.png)

```scala
val spark = SparkSession.builder().appName("hello").master("local[2]").getOrCreate()
val dataframe = spark.read.format("json").load("people.json")
dataframe.printSchema()
dataframe.show(2)
spark.stop()
```

注意：spark.read.format().json()要求json格式为单行，返回的dataframe支持分布式的数据集。和QL类似，有查询、过滤、分组等语句。

```scala
dataframe.select("name","age").show()
dataframe.select(dataframe.col("name"),(dataframe.col("age")+10).as("age2")).show()
dataframe.filter(dataframe.col("age")>20).show()
dataframe.groupBy(dataframe.col("age")).count().show()
```

14、如果分布式结构化数据源没有schema时，即RDD，可以采用RDD转换至dataframe的方式进行数据统计，方法如下：

  a、反射方式（已有的数据源套用在case class模板上）

```scala
val rdd=spark.sparkContext.textFile("people.txt")
import spark.implicits._
val dataframe=rdd.map(_.split(",")).map(line=>Info(line(0).toInt,line(1),line(2).toInt)).toDF()
case class Info(id:Int,name:String,age:Int)
```

另外，如果对之前的dataframe.api操作元数据方式不熟悉的话，可以转换为一张临时表，然后使用QL的方式进行数据统计。

  b、编程方式，同样先获取RDD，然后手动编写schema信息，并最终通过createdataframe将rdd和schema结合。这种方法适用于无法事先得知schema信息。

15、在Spark1.6中，加入了DateSet的支持，基于SQL优化器引擎，可以在编译阶段就能检查出绝大多数错误。

16、Spark1.2中加入了External Dataource API，使得操纵不同存储，不同格式的分布式结构化数据源更加方便。(相较于之前的DataFrame)，最大的特点便是，无需通过sqoop将mysql数据导入到大数据平台，而是可以直接通过spark.read.format("jdbc")进行操纵。 

spark.read.format("jdbc").option("url","jdbc:mysql://localhost:3306").option("dbtable","spark.DEPT").option("password","root").option("driver","com.mysql.jdbc.Driver").load()

17、SparkSQL愿景：

  a、写更少的代码

![image-20200520173330243](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520173439600.png)

右上也是使用RDD，而下方SQL和DF采用更高效的外部结构化数据源。

b、统一接口

![image-20200520173314431](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520173448813.png)

使用sparkcontext统一读写各种格式数据。

  c、schema推导

![image-20200520173448813](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520173455129.png)

![image-20200520173455129](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520173330243.png)

因为有了schema的推导，因此开发人员可以更快速方便的处理不同外部源数据。另外，不同数据源进行schema merge操作是昂贵的，因此默认关闭，需通过.option（"MergeSchema","true"）打开。

  d、分区探测

  当数据源按照字段A分区存储时，传入A的上级目录，可以自动读入所有数据，并按照分区归类。

  e、性能更优

![image-20200520173505066](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520173505066.png)

RDD是基于各自语言的运行环境，且内部无schema信息，相当于当作一个纯文本格式进行操作，而DF由于有了schema信息，且基于内部原生的Spark catalyst，使得运行更高效

  f、读更少的数据

![image-20200520173511285](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520173511285.png)

基于列示存储的DF，可以在读取、统计某些字段信息的时候可以读取更少的数据，并且由于列数据的统一，使得存储空间得到优化。