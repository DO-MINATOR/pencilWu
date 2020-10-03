1、Hive产生背景，由于编写MapReduce过于繁琐，因此诞生了Hive处理，是直接对结构化的数据进行类似于SQL的统计处理得来的。同样建立在Hadoop之上。

2、Hive支持多种执行引擎，例如MR/Spark/Tez等。Hive的功能完成的是从HQL到MR的生成。

3、Hive的体系架构如图所示，client通过shell/jdbc/webui进行操作，Driver完成从HQL到MR的翻译，由于HQL无法直接操作文本文件，因此Hive需要记录每一个文件的元数据信息，借助数据库。

![image-20200519204906707](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519204906707.png)

4、生产环境

![image-20200519204911322](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519204911322.png)

由于Hive只是一个翻译过程，因此没有集群的概念，通常只部署在集群的NameNode上。

5、Hive进行数据离线处理，数据量大，SQL实时处理，数据量小。

6、Hive部署，conf存放配置文件，lib存放jar包（包括mysql-connector等），bin存放启动命令。

![image-20200519204933617](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519204933617.png)

  在hive-env.sh下配置HADOOP_HOME的工作位置，在hive-site.xml下配置mysql-connect的连接信息。

7、通过hive打开shell交互界面

`create database test_db；`创建数据库

`create table line(id int ,name string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'；`创建数据表

`load data local inpath '/home/hadoop/test.txt' overwrite into table line；`将文件格式化到hive仓库（不指明local则为hdfs）

7、通过hive打开shell交互界面

`create database test_db；`创建数据库

`create table line(id int ,name string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'；`创建数据表

`load data local inpath '/home/hadoop/test.txt' overwrite into table line；`将文件格式化到hive仓库（不指明local则为hdfs）

注：读取数据后，源文件将被复制/移动到HDFS的hive数据库目录下。

select count(1) from line;将Hive命令翻译成MR，并通过Yarn进行处理。

Hive与SQL类似，有DDL数据定义语句、DML数据导入导出、DQL数据查询语句。

8、查询操作：hive数据库查询与SQL类似，同为select列名from表名查询条件。

limit用于查询分页。

聚合函数：max（列名），min（列名），avg（列名）等操作。

分组函数：`select deptno，avg(sal) from emp group by deptno；`

`select deptno,job,avg(sal) from emp group by deptno,job；`

`select deptno,avg(sal) avgsal from emp group by deptno having avgsal>=2000；`（注意having而不是where）

多表join函数：`select * from emp e join dept d where e.deptno=d.deptno;`

9、阶段项目，改变用户行为日志分析方式，之前使用MR，如今使用Hive，步骤：

  将ETL数据格式化进Hive中；

  使用QL统计语句进行分析查询。

  这里注意external table和partition的操作。

```sql
create table trackinfo_province(
province string,
cnt bigint)
partitioned by(day string)
row format delimited fields terminated by '\t';指明分区字段。
```

导入数据时也应指明输入的分区。

`load data local inpath '/home/hadoop/data/track.data' overwrite into table trackinfo partition(day='2013-07-21');`

`insert overwrite table trackinfo_province partition(day='2013-07-21') select province,count(1) as cnt from trackinfo where day=('2013-07-21') group by province;`