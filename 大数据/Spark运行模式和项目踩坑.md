1、Spark的4种运行模式

  Local：本地开发测试时使用的模式

  Standalone：如果测试集群环境，则需要每台机器都配置Spark环境

  YARN：借助于已有的hdfs、yarn环境，只需在一台机器配置spark环境即可。

  Mesos

编程代码相同，只是--master提交方式不同。

2、在Yarn环境下分client、cluster模式：

client：Drvier位于client，不能断开连接。日志输出在client控制台

![image-20200520174000729](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520174008002.png)

cluster：Drvier位于ApplicationMaster上，可以断开连接。日志输出在cluster中，需通过yarn application -logs application_id

![image-20200520174008002](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520174000729.png)

3、启用spark on yarn

在spark-env中配置HADOOP_CONF_DIR或者直接在cmd export HADOOP_CONF_DIR

spark-submit --class org.apache.spark.example.SparkPi --master yarn --executor-memory 1G --num-executors 1 /home/hadoop/app/spark-2.1.0-bin-2.6.0/examples/jars/spark-examples_2.11-2.1.-.jar 4

这是client模式，若要使用cluster模式，需要使用--master yarn-cluster

同理spark-shell模式也是如此。

4、将课程统计项目发布至spark-submit上运行，注意需要把scala、spark等jar包设置provided(linux的spark环境以提供了这些jar包)，同时添加

```xml
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <configuration>
        <archive>
            <manifest>
                <mainClass>com.ggstar.util.ip.test</mainClass>
            </manifest>
        </archive>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
</plugin>
```

  a、ipdatabase使用classloader.getResource进行resource目录下的文件读取，但在打包jar之后，会导致目录不正确，因此，应采用classloader.getResourceAsInputStream的方式直接读取文件

  b、使用assembly：assembly进行依赖合并打包，会将不同pom中的可能重名的依赖覆盖，因此在-jar中运行会产生classNoFounder异常，因此应采用maven-shade-plugin，并只需使用package命令进行打包，pom配置如下：

  c、使用spark.read读取parquet文件时，路径务必设置在partition外，否则dataframe将失去partition字段。

  d、链接linux下的数据库，注意url后跟着？charset=utf8

shade方式不会像assembly那样将重名class覆盖打包，使用时直接调用

```xml
<plugin>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.2.0</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <createDependencyReducedPom>false</createDependencyReducedPom>

                <filters>
                    <filter>
                        <artifact>*:*</artifact>
                        <excludes>
                            <exclude>META-INF/*.SF</exclude>
                            <exclude>META-INF/*.DSA</exclude>
                            <exclude>META-INF/*.RSA</exclude>
                        </excludes>
                    </filter>
                </filters>
                <transformers>
                    <transformer
                            implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                    <transformer
                            implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.wsp.log.SparkStatCleanOnYarn</mainClass>
                    </transformer>
                </transformers>

            </configuration>
        </execution>
    </executions>
</plugin>
```

Spark性能调优：

  1、行存储写入性能较高，列存储读取性能较高。

  2、通过spark.sql.parquet.compression.codec指定文件压缩方式，需考虑文件(解)压缩速度，压缩后的分割性。优点：占用空间小、传输速度快。

  3、统计过程使用高性能算子，例如foreachPartition，对于每一条数据，不直接调用connection，而是addbatch。

  4、复用dataframe，因为每一次对dataframe的统计操作都是昂贵的，因此尽可能多的使用公共数据，对公共数据使用df.cache()进行公共复用，最后使用df.unpersist(true)删除cache。

  5、通过spark.sql.shuffle.partitions设置并行度，即task个数，仅yarn模式下有效。

  6、通过spark.sql.sources.partitionColumnTypeInference.enabled设置推测分区字段类型为false，提升效能。

  7、通过accessDF.coalesce(1).write设置写出文件的分块数量。

