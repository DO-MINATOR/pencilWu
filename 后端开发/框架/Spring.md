### 简介

- 轻量级的开源JavaEE框架
- 解决企业开发web工程的复杂性
- 双核心：
  - IOC：控制反转，将对象创建的过程交给Spring管理。
  - AOP：面向切面，减少代码入侵，扩充代码功能。

### 特点

1. 方便解耦，简化开发
2. AOP切面编程
3. 整合Junit方便测试
4. 整合其余框架SpringMVC、Mybatis
5. 封装常用API，如JDBCTemplate、事务
6. 源码精良，适合学习

### 核心包

![image-20210406161233813](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210406161234387.png)

通常为了能正常运行，还需要引入logging日志包。

### IOC

1. 控制反转，将对象创建和之间的依赖调用过程交给框架完成。
2. 通过IOC，降低了非业务代码的编写，降低模块间的耦合度。

#### 原理

- xml解析
- 工厂模式、单例模式
- 反射

**图示过程**

在userService中调用userDao的方法。需要尽可能降低模块之间的耦合度

1. 原始方式
   ![image-20210406162738027](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210406162738027.png)

2. 工厂模式
   ![image-20210406162806623](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210406162806623.png)
   此时仍存在模块间的耦合，由于在factory中对UserDao的限制过于明确，此时Dao的修改会引起Factory的修改。

3. xml解析+工厂+反射
   ![image-20210406163608980](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210406163608980.png)

   此时只需要修改xml中class的全限定名即可实现具体模块的替换。

IOC思想基于容器来完成，底层就是对象工厂。

提供了两个实现方式：

1. BeanFactory：开发人员一般不使用。对象在使用时才创建。
2. ApplicationContext：BeanFactory的子接口，功能更强大。加载配置文件随即创建对象，需要开启单例模式。

```java
@Test
public void test1(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("Spring.xml");
    User user = applicationContext.getBean("user", User.class);
    user.add();
}
```

#### Bean管理

**xml**

```xml
<bean id="user" class="...">
```

id：获取对象的标识
class：类的全限定名
创建对象默认调用无参构造方法。
DI：依赖注入的具体实现。（注意循环依赖的问题，属性自动注入可解决，但构造器中依赖无法解决，参加IOC的三级缓存管理）

```xml
<bean id="service" class="...">
    <property name="name" value="hello"></property>
    <property name="userdao" ref="daoImpl"></property>
</bean>
```

数组、List、Map和Set集合类属性的注入，注意，对于集合类，Spring有它默认的实现类，比如List是LinkedList，Map是LinkedHashMap。

**注解**

相对xml更加简洁。

1. 引入依赖spring-aop

2. 开启组件扫描

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"#开启组件扫描
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
   
       <context:component-scan base-package="service"></context:component-scan><!--待扫描的包-->
   </beans>
   ```

3. 依赖注入

   - 对象类型注入：@Autowired（根据类型）、@Qualifier（根据名称，需与autowired一起使用）、@Resource（前两者的混合体）
   - 普通类型注入：@Value

#### 生命周期

1. 调用对象的构造器方法
2. 设置对象依赖
3. 调用bean的初始化方法（需要手动配置）
4. 使用bean
5. 容器关闭时，调用bean的销毁方法（需要手动配置）

### xml管理

如果将所有的bean管理（对象创建和依赖注入）全部放到application.xml中，就会繁杂，因此类似于jdbc.properties的方式引入外部配置文件，并在该xml中导入。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                   http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="person" class="service.Person">
        <property name="name" value="${name}"></property>
    </bean>
    <context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>
</beans>
```

### AOP

不改变原有代码的基础上，增强原有方法。

```xml
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

#### 原理

- 有接口的情况，JDK动态代理，创建的是接口的实现类作为代理对象，使用Proxy.newProxyInstance()创建代理对象，该方法有三个参数，分别是Classloader loader、Class<?>[] interfaces、InvocationHandler h，即动态生成接口class结构，并用和被代理对象相同的类加载器加载该class文件，这样相当于创建了该接口下的实例对象，调用同名方法后，内部调用h的invoke方法。invoke才是增强方法的精髓。
  ![image-20210407211153062](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210407211153062.png)
- 没有接口的情况，CGLIB动态代理，创建的是被代理对象的子类作为代理对象
  ![image-20210407211312227](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210407211312227.png)

#### 术语

- 连接点：可以被代理的方法
- 切入点：实际被代理的方法
- 通知（增强）：代理对象增加的方法
  - 前置
  - 后置
  - 环绕
  - 异常
  - 最终：类似于finally
- 切面：切入点+通知

AspectJ，一个独立的jar包，通常和Spring一起实现AOP，可以通过xml、注解。

#### 切入点表达式

execution([权限修饰符]、[返回类型]、[类全限定名]、[方法名]、[参数列表])

- 对com.wsp.dao.Userdao的add方法进行增强

  ```
  execution(* com.wsp.dao.Userdao.add(..))
  ```

- 对com.wsp.dao.Userdao的所有方法进行增强

  ```
  execution(* com.wsp.dao.Userdao.*(..))
  ```

- 对com.wsp.dao包下所有类的所有方法进行增强

  ```
  execution(* com.wsp.dao.*.*(..))
  ```

注意权限修饰符一般不限定(*)，返回类型通常可以省略。

### JDBCTemplate

Spring对JDBC的二次封装，首先是xml配置文件。

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
     <property name="url" value="jdbc:mysql:///user_db" />
     <property name="username" value="root" />
     <property name="password" value="root" />
     <property name="driverClassName" value="com.mysql.jdbc.Driver" />
</bean>

<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
	<property name="dataSource" ref="dataSource"></property>
</bean>

<context:component-scan base-package="com.atguigu"></context:component-scan>
```

```java
@Repository
public class BookDaoImpl implements BookDao {
     //注入 JdbcTemplate
     @Autowired
     private JdbcTemplate jdbcTemplate;
}
```

增删改操作：

jdbctemplate的update(String sql, Object... args)

```java
public void add(Book book) {
    //1 创建 sql 语句
    String sql = "insert into t_book values(?,?,?)";
    //2 调用方法实现
    Object[] args = {book.getUserId(), book.getUsername(), book.getUstatus()};
    int update = jdbcTemplate.update(sql,args);
    System.out.println(update);
}
```

查询操作：
- 单个数量值的查询。

  ```java
  public int selectCount() {
  	String sql = "select count(*) from t_book";
  	Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
  	return count;
  }
  ```

- 单个bean对象的查询。

  ```java
  public Book findBookInfo(String id) {
       String sql = "select * from t_book where user_id=?";
       //调用方法
       Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Book>(Book.class), id);//BeanPropertyRowMapper用于将查询结果封装为单个bean对象
       return book;
  }
  ```

- 多个bean对象list的查询。

  ```java
  public List<Book> findAllBook() {
       String sql = "select * from t_book";
       //调用方法
       List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class));//BeanPropertyRowMapper用于将查询结果封装为单个bean对象并添加到List中。
       return bookList;
  }
  ```

批量增删改操作：

jdbctemplate的batchUpdate(String sql, List<Object[]> batchargs)

```java
public void batchAdd(List<Object[]> batchArgs) {
     String sql = "insert into t_book values(?,?,?)";
     int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
     System.out.println(Arrays.toString(ints));
}
//批量添加测试
List<Object[]> batchArgs = new ArrayList<>();
Object[] o1 = {"3","java","a"};
Object[] o2 = {"4","c++","b"};
Object[] o3 = {"5","MySQL","c"};
batchArgs.add(o1);
batchArgs.add(o2);
batchArgs.add(o3);
//调用批量添加
bookService.batchAdd(batchArgs);
```

### 事务

编程式事务管理，即通过手动开启事务，提交事务或回滚事务完成原子性、一致性管理。声明式事务管理，通过xml或注解方式进行管理，底层是通过AOP实现事务管理。

xml中配置事务管理

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
 <!--注入数据源-->
     <property name="dataSource" ref="dataSource"></property>
</bean>
<!--引入名称空间tx-->
xmlns:tx="http://www.springframework.org/schema/tx"
<!--开启事务注解-->
<tx:annotation-driven transactionmanager="transactionManager"></tx:annotation-driven>
```

在service中对应方法或类上直接添加@Transactional注解表示开启事务操作。

**传播行为propagation**

|     属性     |                          描述                          |
| :----------: | :----------------------------------------------------: |
|   Required   | 如果有事务，则在当前事务中进行，否则新开启一个事务执行 |
| Required_New |          无论是否有事务，均开启一个新事务执行          |
|    Never     |              当前方法不应运行在任何事务中              |

**隔离级别isolation**

- read_uncommitted
- read committed
- repeatable read
- serializable

**超时时间timeout**

事务需在一定时间内进行提交，否则进行回滚，默认-1，表示不计时。

**是否只读readonly**

设置为true，则无法进行增删改操作。

**异常回滚rollbackFor**

设置出现哪些异常进行回滚。

**注意：**

1. 上述属性的设置如果传播级别是Required，则只和父级的属性设置有关。
2. 如果是类中方法互相调用，则无法起到事务属性重新设置的作用，因为底层是通过AOP进行动态代理创建的Proxy对象，如果是类中方法调用则起不到动态代理作用，相当于只是被代理对象中方法的普通调用。

![image-20210412222115454](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210412222115454.png)