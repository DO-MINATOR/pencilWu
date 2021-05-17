### 简介

- 将重要步骤提取至配置文件中，方便维护
- 将业务逻辑和sql分离，做到sql优化
- mybatis底层就是对原生JDBC的简单封装

### 步骤

1. 全局配置文件：mybatis-config.xml，如连接信息，事务控制，通过properties引入jdbc连接信息

   ```xml
   <configuration>
       <properties resource="jdbc.properties"></properties>
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="${driver}"/>
                   <property name="url" value="${url}"/>
                   <property name="username" value="${username}"/>
                   <property name="password" value="${password}"/>
               </dataSource>
           </environment>
       </environments>
       <mappers>
           <mapper resource="SeatDao.xml"/>
       </mappers>
   </configuration>
   ```

2. SQL映射文件：EmployeeDao.xml，对Dao接口的具体实现描述，mybatis通过该配置信息创建dao的具体实现类。

   ```xml
   <mapper namespace="com.wsp.dao.SeatDao">
       <select id="getById" resultType="com.wsp.bean.Seat"><!--注意使用类的全限定名-->
       select * from test where id = #{id}
     </select>
       <delete id="delete">
           delete from test where id=#{id}
       </delete>
       <insert id="insert">
           insert into test(id,seat) values(#{id},#{seat})
       </insert>
       <update id="update">
           update test set seat=#{seat} where id=#{id}
       </update>
   </mapper>
   ```

3. 创建sqlsessionfactory

4. 得到sqlsession。

5. 执行方法，返回bean/常数

### 参数获取

- 传入一个参数，#{任意变量名均可取出}

- 传入多个参数，自动封装为key='param1',value=Object的map，所以只能通过#{param1}获取第一个参数，可以通过给dao方法添加注解设置key的别名，如

  ```java
  public int deletebyIdAndName(@Param("id") int id,@Param("name") String name);
  ```

- 传入pojo对象，通过#{属性名}获取

- 传入map对象，通过#{key}获取

**注意：**#{}是通过预编译执行参数取值的，而${}通过拼串方式，前者可防止SQL注入，当无法进行参数取值时，使用\${}获取，如设置参数查询不同表。

### 查询返回集合

```xml
<select id="getAll" resultType="com.wsp.bean.Seat">
    select * from test
</select>
```

resutlType写List中泛型的类全限定名

### 自定义封装resultMap

```xml
<select id="getAll" resultMap="mymap1">
    select * from test
</select>
<resultMap type="com.wsp.bean.Seat" id="mymap1">
    <id column="id" property="id"></id><!--主键id-->
    <result column="wasd" property="ddd"></result><!--普通属性result-->
</resultMap>
```

默认的封装规则是将列名不区分大小写的直接与属性名对应，也可以将下划线和开启驼峰命名法相匹配。其他情况下除了使用别名外，resultMap可以自定义列名和属性名的映射关系。此外，这种方式还支持级联赋值，如果bean对象中包含了另一个bean，则在result的property中使用bean.***进行赋值关联。

不过Mybatis推荐使用\<association>进行级联赋值，并使用javaType指定类型。
对于级联属性是List集合这种，通常使用\<collection>，并用ofType指定泛型类型。

学生多对一老师查询案例：

```xml
<mapper namespace="com.wsp.dao.TeacherDao">
    <select id="getTeacherById" resultMap="map1">
        select t.* ,s.id sid ,s.name sname from teacher t left join student s on s.t_id=t.id where t.id=#{id}
    </select>
    <resultMap id="map1" type="com.wsp.bean.Teacher">
        <id column="id" property="id"></id>
        <result column="name" property="name"></result>
        <collection property="list" ofType="com.wsp.bean.Student">
            <id column="sid" property="id"></id>
            <result column="sname" property="name"></result>
        </collection>
    </resultMap>
</mapper>
```

**注意：**java bean对象最好带上无参构造器，否则容易出现bug

### 分步查询

在一些比较复杂的业务场景下，还可以使用分步查询，即调用mapper接口最终查询的是多个表的字段，并一同封装进去。

```xml
<select id="getTeacherByIdSimple" resultMap="map2">
    select * from teacher where id =#{id}
</select>
<resultMap id="map2" type="com.wsp.bean.Teacher">
    <id property="id" column="id"></id>
    <result property="name" column="name"></result>
    <collection property="list" column="id" select="com.wsp.dao.StudentDao.getStudentBytIdSimple">
        <!--select * from student where t_id = #{tid}-->
    </collection><!--将id字段作为参数传入到另一个dao方法中进行二次查询，并将结果封装为list保存到teacher中的list属性-->
</resultMap>
```

同时可以开启延迟加载提高效率。

### 动态SQL

**\<if test>**条件拼串：

```xml
<!--    public List<Student> getStudentByStudent(Student student);-->
<select id="getStudentByStudent" resultType="com.wsp.bean.Student">
    select id,name from student
    <where>
        <if test="id!=null and id!=0">
            id=#{id}
        </if>
        <if test="name!=null and !&quot;&quot;.equals(name)">
            and name like#{name}<!--注意<where>中and写在前面，这样即使前面不满足，紧接的and也会被去除掉-->
        </if>
    </where> 
</select>
```

```java
student.setName("%a%");
List<Student> studentByStudent = mapper.getStudentByStudent(student);
```

**\<foreach>**遍历：

该标签主要用于当传入的参数是list或map时

```xml
<!--    public List<Student> getStudentIn(List<Student> slist);-->
<select id="getStudentIn" resultType="com.wsp.bean.Student">
    select id,name from student where id in
    <foreach collection="slist" open="(" close=")" item="item" separator=",">
        #{item.id}
    </foreach>
</select>
```

**\<choose>单分支选择**：

类似if test，不过只会进入一个分支中

**set结合if标签实现部分字段更新：**

```xml
<!--    public void updateStudent(Student student);-->
<update id="updateStudent">
    update student
    <set>
        <if test="name!=null and !&quot;&quot;.equals(name)">
            name = #{name},
        </if>
        <if test="tid!=0">
            t_id=#{tid}
        </if>
    </set>
    <where>
        id=#{id}
    </where>
</update>
```

### 缓存

- 一级缓存：同一个sqlsession下共享的数据，默认启用。只要之前查询过的数据，会保存在map中，下次直接获取。
- 二级缓存：不同session共享的数据，需手动开启，且只有当session关闭后才会放入缓存。

只要执行了一次增删改操作，都会清除一级缓存。也可以手动清空，`sqlSession.clearCache()`

二级缓存测试

```xml
<configuration>
    <settings>
        <setting name="cacheEnabled" value="true"/>
        <setting name="logImpl" value="STDOUT_LOGGING"/><!--开启log4j日志记录-->
    </settings>
    ...
</configuration>
```

log4j.properties

```properties
log4j.rootLogger=DEBUG,console,FILE

log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.threshold=INFO
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} [%5p] - %c -%F(%L) -%m%n

log4j.appender.FILE=org.apache.log4j.RollingFileAppender
log4j.appender.FILE.Append=true
log4j.appender.FILE.File=logs/log4jtest.log
log4j.appender.FILE.Threshold=INFO
log4j.appender.FILE.layout=org.apache.log4j.PatternLayout
log4j.appender.FILE.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} [%5p] - %c -%F(%L) -%m%n
log4j.appender.FILE.MaxFileSize=10MB
```

studentDao.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.wsp.dao.StudentDao">
    <cache></cache>
	...
</mapper>
```

**注意：**若想成功实现二级缓存，student还需要implements Serializable，以及第一个sqlsession需要close。

```java
SqlSession sqlSession = sqlSessionFactory.openSession();
StudentDao mapper = sqlSession.getMapper(StudentDao.class);
List<Student> studentBytIdSimple = mapper.getStudentBytIdSimple(1);
for (Student student : studentBytIdSimple) {
    System.out.println(student);
}
sqlSession.close();
SqlSession sqlSession2 = sqlSessionFactory.openSession();
StudentDao mapper2 = sqlSession2.getMapper(StudentDao.class);
List<Student> studentBytIdSimple2 = mapper2.getStudentBytIdSimple(1);
for (Student student : studentBytIdSimple2) {
    System.out.println(student);
}
```

![image-20210505103231062](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210505103231062.png)

mybatis提供了一个缓存实现接口，可自己提供第三方缓存实现。

### 延迟加载

当使用\<association>查询关联对象时，可以做到属性按需加载，即java代码如果访问到了关联的属性时，才会去查询。开启方法是在mybatis.config中配置

```xml
<settings>
    <setting name="LazyLoadingEnabled" value="true"></setting>
    <setting name="aggressiveLazyLoading" value="false"></setting>
</settings>
```

### MyBatis逆向

正常的mybatis创建流程是`javabean、table、dao.java、dao.xml`

MBG的思想是在只给出table的情况下，自动生成javabean、dao接口和daomap查询

**配置：**

1. \<jdbcconnection>配置数据库连接信息
2. \<javamodelGenerator>配置创建出bean对象所在的目录和包路径
3. \<sqlMapGenerator>配置map查询所在的目录和包路径
4. \<javaClientGenerator>配置dao接口生成的目录和包路径
5. \<table>配置table表和bean对象的映射关系

可通过命令行或java代码进行生成（查阅官方文档）

