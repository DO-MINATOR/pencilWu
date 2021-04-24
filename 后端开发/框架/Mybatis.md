1、Mybatis的使用需要引入jar包Mybatis，配置文件Configuration.xml和Message.xml，前者是配置底层数据库驱动（JDBC）以及链接URL、用户名和密码等，后者配置数据库语句、ORM映射关系等。

2、获取数据库session对象是通过SqlSessionFactoryBuilder并传入配置文件所生成的，在sqlsession执行delete等数据库修改操作以后，是需要进行提交的commit。

3、如果配置好映射关系以后，session查询所返回的结果会默认封装成bean对象的，查询多条则用List包裹起来。

4、对数据库执行语句进行传参，需要指明参数对象parameterType，且只能有一个，如果是普通变量，则直接指明（int、String），对象需要使用完整包名指出，List使用List，Map使用Map；对参数内部的访问，普通变量直接使用#{_parameter}，对象直接使用#{属性名 }，list使用\<foreach collection="list" item="item">

5、此外mybatis还有配置一对多关系。

6、Mybaits的MapperScannerConfigurer可以自动扫描Dao接口，并生成代理类，且自动打上注解（@Repository），代理类执行增删改查方法时，会去mapper.xml中扫描是否有对应方法，且参数、返回值须一致。

### 简介

- 将重要步骤提取至配置文件中·，方便维护
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
           <mapper resource="SeatDao.xml"/><!--注意该文件命名需要和dao接口类名称一致-->>
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

