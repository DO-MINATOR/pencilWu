1、Hadoop提供分布式的存储（拆分为很多块，并且以副本的形式存储各台机器）和计算。使得为用户屏蔽底层实现细节（好比在一台机器上进行操作）广义的讲是一个涵盖多种对应框架的集合。不止HDFS、MapReduce、YARN。

2、核心模块：

  HDFS分布式文件存储系统：扩展性/容错性/海量存储(Hadoop Distributed Filesystem)

![image-20200519201814596](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519201814596.png)

MapReduce分布式计算框架：扩展性/容错性/离线处理

![image-20200519201833496](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519201833496.png)

YARN集群资源的管理和调度：扩展性/容错性/多框架统一调度（Yet another resource negotiator）另一种资源调度者，相对dfs的文件资源调度管理

![image-20200519202048920](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519202121563.png)

3、HDFS的架构，包含一个NameNode和多个DataNodes，NameNode负责对外提供文件的访问（CRUD）接口，典型应用包括对一个文件进行Block切分、存放、记录Block对应的DataNodes的映射，而不同的DataNodes负责执行具体操作。相较于传统的文件系统，HDFS更加注重文件的流式传输以及高容错性。

4、OOTB环境搭建（立即可用的）

![image-20200519202103718](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519202048920.png)

5、由于Hadoop基于java开发，所以需要有JDK运行环境。NameNode和DataNodes之间需要进行数据传递，为了能够进行主从节点间的通信，需要配置ssh无密码登录。

使用ssh-keygen -t rsa生成A主机的公私钥，登录到B主机，cat id_ras.pub >> authorized_keys，今后A可以无需密码登录到B。须要对keys文件改权限为600。

6、下载并解压hadoop

![image-20200519202113012](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519202127795.png)

bin为客户端命令，sbin启动服务，etc/hadoop为配置文件

lib为相关jar包，share常用例子

7、配置etc/hadoop/hadoop-env.sh，配置java实现，同样还需配置环境变量。

![image-20200519202121563](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519202141539.png)

配置core-site.xml，配置NameNode对外的ip地址和端口号。

![image-20200519202127795](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519202113012.png)

配置hdfs-site.xml，单机环境下副本计数为1即可，文件block存放位置

![image-20200519202134403](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519202150747.png)

配置DataNodes，向 slaves添加hadoop000,即告知NameNode有哪些DataNodes.

首次执行hdfs需要格式化文件系统，hdfs namenode -format 

![image-20200519202141539](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519202134403.png)

8、启动服务./sbin/start-dfs.sh，使用jps（java运行程序查看器），如果出现三个node说明启动成功，通过http://192.168.1.233:9000/访问文件系统，需要注意防火墙。

![image-20200519202150747](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519202103718.png)

9、hadoop常用命令：

hadoop fs（启动一个文件系统客户端）：

-ls 列举hdfs里的文件

-put、-copyFromLocal 拷贝至hdfs

-cat、-text 显示文件内容

-moveFromLocal 移动至hdfs

-get、-copyToLocal 从hdfs获取文件

-mkdir 创建文件夹

-mv 移动、改名文件

-getmerge 获取文件并合并

-rm 删除文件

-rm -r 删除文件夹

例举：将一个173MB的文件put至HDFS，NameNode会自动进行计数块拆解，并记录存放的DataNodes，get文件时，NameNode会自动对文件进行getmerge操作。

10、IDEA开发HDFS的API版本，在maven引入CDH的仓库，然后引入依赖

```xml
<repositories>
    <repository>
      <id>cloudera</id>
      <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
    </repository>
    
  </repositories>
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-client</artifactId>
      <version>${hadoop.version}</version>
    </dependency>
```

11、API接口开发步骤：

  1、设置配置configuration，如果不指定则默认副本计数为3，调用configuration.set("dfs.replication", "1")以修改。

  2、FileSystem.get(new URI(".."), configuration, "用户名");

常用的接口函数：

  mkdir、open（获取文件输入流）、create（创建文件输出流）、rename、copyFromLocal、listStatus（获取Path下文件信息）、listFiles（可迭代获取）、getFileBlockLocations获取文件块信息、delete

12、阶段项目之HDFS文件词频统计

  配置文件的读取可以采用

```java
Properties properties = new Properties();
properties.load(ParamsUtils.class.getClassLoader().getResourceAsStream("wc.properties"));
```

除了面向接口编程，还可以采用反射的机制（参考IOC）

13、副本摆放机制

  通常DataNodes会分布在不同的rack（机架）上，rack内部的节点传输速度快于rack间，然而不会将所有block全部存放在同一rack，因为这样容错性不高。常规的副本计数为3，摆放策略有

B1：rack1 B1'：rack2 B1''：rack2、B1：rack1 B1'：rack1 B1''：rack2

14、客户端向DFS发送文件过程

NameNode只是作为一个文件传输请求的应允者和文件元数据信息的记录者。而DataNodes才是真正进行数据信息操作的节点。

![image-20200519202310442](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519202326423.png)

15、客户端向DFS请求读数据

![image-20200519202317663](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519202317663.png)

16、NameNode的文件元数据信息相当于是目录结构，保存着文件Id、副本计数、块存节点，然后客户端频繁的提交dfs指令，当元数据信息是以文件的形式存放在NameNode磁盘中时，会导致IO忙，因此元数据应是存放在内存中，并且进行定期保存（序列化），同时会时刻的进行dfs指令记录。

如图，SecondaryNameNode负责从NameNode拉取目录结构快照、操作记录，然后进行目录结构跟踪，并将新的目录结构快照推送到NameNode磁盘中，当NameNode宕机后，可以快速进行文件元数据的复原。这就是HDFS的checkpoint机制。

![image-20200519202326423](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519202310442.png)

17、HDFS的safemode，当hdfs启动时，会进行大致30秒的block检查，检查所有DataNodes汇报的块信息，（如果文件元数据信息记录当前有10个节点，当datanodes汇报的block数量达到10的阈值%时，将退出safenmode）。注意，在safemode下，hdfs无法进行任何文件读写操作，这是一种系统保护机制。	