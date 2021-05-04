1、Mybatis的使用需要引入jar包Mybatis，配置文件Configuration.xml和Message.xml，前者是配置底层数据库驱动（JDBC）以及链接URL、用户名和密码等，后者配置数据库语句、ORM映射关系等。

2、获取数据库session对象是通过SqlSessionFactoryBuilder并传入配置文件所生成的，在sqlsession执行delete等数据库修改操作以后，是需要进行提交的commit。

3、如果配置好映射关系以后，session查询所返回的结果会默认封装成bean对象的，查询多条则用List包裹起来。

4、对数据库执行语句进行传参，需要指明参数对象parameterType，且只能有一个，如果是普通变量，则直接指明（int、String），对象需要使用完整包名指出，List使用List，Map使用Map；对参数内部的访问，普通变量直接使用#{_parameter}，对象直接使用#{属性名 }，list使用\<foreach collection="list" item="item">

5、此外mybatis还有配置一对多关系。

6、Mybaits的MapperScannerConfigurer可以自动扫描Dao接口，并生成代理类，且自动打上注解（@Repository），代理类执行增删改查方法时，会去mapper.xml中扫描是否有对应方法，且参数、返回值须一致。

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

### Dao.xml

```xml
<mapper namespace="com.wsp.dao.SeatDao"> <!--指定某接口-->
    <select id="getById" resultType="com.wsp.bean.Seat">
    select * from test where id = #{id}
    </select>
    <delete id="delete">
        delete from test where id=#{id}
    </delete>
    <insert id="insert" useGeneratedKeys="true" keyProperty="id"> <!--将自增主键自动赋值给传入的对象-->
        insert into test(id,seat) values(#{id},#{seat})
    </insert>
    <update id="update">
        update test set seat=#{seat} where id=#{id}
    </update>
</mapper>
```

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
- 二级缓存：不同session共享的数据

只要执行了一次增删改操作，都会清除一级缓存。也可以手动清空，`sqlSession.clearCache()`

