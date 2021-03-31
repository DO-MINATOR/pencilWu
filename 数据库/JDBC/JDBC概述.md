### 持久化简介

- 持久化：把数据保存到可掉电式存储设备中以供之后使用。

- 持久化的主要应用是将内存中的数据存储在关系型数据库中，当然也可以存储在磁盘文件、XML数据文件中。

![1566741430592](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/1566741430592.png)

### Java存储技术

- 在Java中，数据库存取技术可分为如下几类：
  - **JDBC**直接访问数据库
  - JDO (Java Data Object )技术

  - **第三方ORM工具**，如Hibernate, Mybatis 等

- JDBC是java访问数据库的基石，JDO、Hibernate、MyBatis等只是更好的封装了JDBC。

### JDBC简介

- JDBC(Java Database Connectivity)是一个**独立于特定数据库管理系统、通用的SQL数据库存取和操作的公共接口**（一组API），定义了用来访问数据库的标准Java类库，（**java.sql,javax.sql**）使用这些类库可以以一种**标准**的方法、方便地访问数据库资源。
- JDBC为访问不同的数据库提供了一种**统一的接口**，为开发者屏蔽了一些细节问题。
- JDBC的目标是使Java程序员使用JDBC可以连接任何**提供了JDBC驱动程序**的数据库系统，这样就使得程序员无需对特定的数据库系统的特点有过多的了解，从而大大简化和加快了开发过程。

![1555575941569](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/1555575941569.png)

### 执行步骤

![1565969323908](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/1565969323908.png)

