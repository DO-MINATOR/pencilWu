### ShardingSphere

ShardingSphere是一个支撑关系型数据库的数据分片中间件，该生态包含三个主要产品，分别是ShardingJDBC、ShardingProxy和ShardingSidecar。

其中jdbc作用于客户端，进一步封装Driver，根据一定规则帮助客户端实现读写分离、分片操作。而proxy相当于一个代理中间件，可以兼容任何支持MySQL协议的客户端。

 很显然，ShardingJDBC只是客户端的一个工具包，可以理解为一个特殊的JDBC驱动包，所有分库分表逻辑均由业务方自己控制，所以他的功能相对灵活，支持的数据库也非常多，但是对业务侵入大，需要业务方自己定制所有的分库分表逻辑。而ShardingProxy是一个独立部署的服务，对业务方无侵入，业务方可以像用一个普通的MySQL服务一样进行数据交互，基本上感觉不到后端分库分表逻辑的存在，但是这也意味着功能会比较固定，能够支持的数据库也比较少。这两者各有优劣。

### 相关概念

- 逻辑表：水平拆分的数据库的相同逻辑和数据结构表的总称
- 真实表：在分片的数据库中真实存在的物理表。
- 数据节点：数据分片的最小单元。由数据源名称和数据表组成
- 绑定表：分片规则一致的主表和子表。
- 广播表：也叫公共表，指素有的分片数据源中都存在的表，表结构和表中的数据在每个数据库中都完全一致。例如字典表。
- 分片键：用于分片的数据库字段，是将数据库(表)进行水平拆分的关键字段。SQL中若没有分片字段，将会执行全路由，性能会很差。
- 分片算法：通过分片算法将数据进行分片，支持通过=、BETWEEN和IN分片。分片算法需要由应用开发者自行实现，可实现的灵活度非常高。
- 分片策略：真正用于进行分片操作的是分片键+分片算法，也就是分片策略。在ShardingJDBC中一般采用基于Groovy表达式的inline分片策略，通过一个包含分片键的算法表达式来制定分片策略，如t_user_$->{u_id%8}标识根据u_id模8，分成8张表，表名称为t_user_0到t_user_7。

### 基础分片算法inline

通过maven引入mybatis、shardingsphere、junit测试等。

application.properties

```properties
#垂直分表策略
# 配置真实数据源
spring.shardingsphere.datasource.names=m1

# 配置第 1 个数据源
spring.shardingsphere.datasource.m1.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.m1.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.m1.url=jdbc:mysql://localhost:3306/coursedb?serverTimezone=GMT%2B8
spring.shardingsphere.datasource.m1.username=root
spring.shardingsphere.datasource.m1.password=root

# 指定表的分布情况 配置表在哪个数据库里，表名是什么。水平分表，分两个表：m1.course_1,m1.course_2
spring.shardingsphere.sharding.tables.course.actual-data-nodes=m1.course_$->{1..2}

# 指定表的主键生成策略
spring.shardingsphere.sharding.tables.course.key-generator.column=cid
spring.shardingsphere.sharding.tables.course.key-generator.type=SNOWFLAKE
#雪花算法的一个可选参数
spring.shardingsphere.sharding.tables.course.key-generator.props.worker.id=1

#指定分片策略 约定cid值为偶数添加到course_1表。如果是奇数添加到course_2表。
# 选定计算的字段
spring.shardingsphere.sharding.tables.course.table-strategy.inline.sharding-column= cid
# 根据计算的字段算出对应的表名。
spring.shardingsphere.sharding.tables.course.table-strategy.inline.algorithm-expression=course_$->{cid%2+1}

# 打开sql日志输出。
spring.shardingsphere.props.sql.show=true
spring.main.allow-bean-definition-overriding=true
```

之后执行addcourse测试代码。

![image-20211206200911412](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211206200911412.png)

可以看到控制台输出如下语句。

```conf
2020-12-15 18:35:16.426  INFO 22412 --- [           main] ShardingSphere-SQL                       : Logic SQL: INSERT INTO course  ( cname,user_id,cstatus )  VALUES  ( ?,?,? )
2020-12-15 18:35:16.427  INFO 22412 --- [           main] ShardingSphere-SQL                       : Actual SQL: m1 ::: INSERT INTO course_2  ( cname,user_id,cstatus , cid)  VALUES  (?, ?, ?, ?) ::: [java, 1001, 1, 545674405561237505]
```

从这个日志中我们可以看到，程序中执行的Logic SQL经过ShardingJDBC处理后，被转换成了Actual SQL往数据库里执行。执行的结果可以在MySQL中看到，course_1和course_2两个表中各插入了五条消息。这就是ShardingJDBC帮我们进行的数据库的分库分表操作。

![image-20211206201126842](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211206201126842.png)

### 高级分片算法

 ShardingJDBC的整个实战完成后，可以看到，整个分库分表的核心就是在于配置的分片算法。我们的这些实战都是使用的inline分片算法，即提供一个分片键和一个分片表达式来制定分片算法。这种方式配置简单，功能灵活，是分库分表最佳的配置方式，并且对于绝大多数的分库分片场景来说，都已经非常好用了。但是，如果针对一些更为复杂的分片策略，例如多分片键、按范围分片等场景，inline分片算法就有点力不从心了。

常用的五种分片策略：

1. NoneShardingStrategy

   不分片。这种严格来说不算是一种分片策略了。只是ShardingSphere也提供了这么一个配置。

2. InlineShardingStrategy

   最常用的分片方式

   - 配置参数： inline.shardingColumn 分片键；inline.algorithmExpression 分片表达式
   - 实现方式： 按照分片表达式来进行分片。

3. StandardShardingStrategy

   只支持单分片键的标准分片策略。

   - 配置参数：standard.sharding-column 分片键；standard.precise-algorithm-class-name 精确分片算法类名；standard.range-algorithm-class-name 范围分片算法类名

   - 实现方式：

     shardingColumn指定分片键。

     preciseAlgorithmClassName 指向一个实现了io.shardingsphere.api.algorithm.sharding.standard.PreciseShardingAlgorithm接口的java类名，提供按照 = 或者 IN 逻辑的精确分片 `示例：com.roy.shardingDemo.algorithm.MyPreciseShardingAlgorithm`

     rangeAlgorithmClassName 指向一个实现了 io.shardingsphere.api.algorithm.sharding.standard.RangeShardingAlgorithm接口的java类名，提供按照Between 条件进行的范围分片。`示例：com.roy.shardingDemo.algorithm.MyRangeShardingAlgorithm`

   - 说明：

     其中精确分片算法是必须提供的，而范围分片算法则是可选的。

4. ComplexShardingStrategy

   支持多分片键的复杂分片策略。

   - 配置参数：complex.sharding-columns 分片键(多个); complex.algorithm-class-name 分片算法实现类。

   - 实现方式：

     shardingColumn指定多个分片键。

     algorithmClassName指向一个实现了org.apache.shardingsphere.api.sharding.complex.ComplexKeysShardingAlgorithm接口的java类名。提供按照多个分片列进行综合分片的算法。`示例：com.roy.shardingDemo.algorithm.MyComplexKeysShardingAlgorithm`

     多键的类型需要统一。

5. HintShardingStrategy

   不需要分片键的强制分片策略。这个分片策略，简单来理解就是说，他的分片键不再跟SQL语句相关联，而是用程序另行指定。对于一些复杂的情况，例如select count(*) from (select userid from t_user where userid in (1,3,5,7,9)) 这样的SQL语句，就没法通过SQL语句来指定一个分片键。这个时候就可以通过程序，给他另行执行一个分片键，例如在按userid奇偶分片的策略下，可以指定1作为分片键，然后自行指定他的分片策略。

   - 配置参数：hint.algorithm-class-name 分片算法实现类。

   - 实现方式：

     algorithmClassName指向一个实现了org.apache.shardingsphere.api.sharding.hint.HintShardingAlgorithm接口的java类名。 `示例：com.roy.shardingDemo.algorithm.MyHintShardingAlgorithm`

     在这个算法类中，同样是需要分片键的。而分片键的指定是通过HintManager.addDatabaseShardingValue方法(分库)和HintManager.addTableShardingValue(分表)来指定。

     使用时要注意，这个分片键是线程隔离的，只在当前线程有效，所以通常建议使用之后立即关闭，或者用try资源方式打开。



