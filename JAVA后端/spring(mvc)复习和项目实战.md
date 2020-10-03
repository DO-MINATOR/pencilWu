1、spring为EE提供了一站式解决方案（包括web层，业务层，实体层，Dao层等）

2、spring的优点包括：解耦，方便开发，便于维护（将对象的创建和依赖交给spring去维护，实现解耦功能）；支持AOP编程；支持事务管理；集成junit4、Hibernate、Mybaits等框架。

3、open-close（开闭原则）即对源码修改关闭，对扩展开放，具体的实现方式为针对抽象类编写不同的业务实现类，而不是只对一个实现类不停的进行修改。

4、springBean管理两种方式:

  A、xml配置：其中，优先采用bean的无参构造方法，其次使用静态工厂或者实例工厂的方法获取相关bean（工厂模式，只关心如何获取一个具体实现类，具体类的逻辑设计不关心，符合开闭原则）。后二者方式如下

```xml
<bean id="be" class="com.beanfactory" factory-method="getbean"></bean>
<bean id="bf" class="com.beanfactory"></bean>
<bean id="be" factory-bean="bf" factory-method="getbean"></bean>
```

scope作用域：singleton（单例模式）、prototype（每次getbean返回一个新的实例）、request（每次请求返回一个实例）

生命周期：初始化在构造方法后执行，销毁在context销毁时执行(singleton有效)

属性注入：构造方法注入，使用<constructor-arg name value>；set方法注入，使用<property name value>，对象的话value替换为ref；p名称空间的属性注入也适用于set方法，相比property更加简洁

复杂类型属性注入：如果有array、list（使用list+value标签注入）set（set+value注入）、map（map+entry（key+value）注入）

B、注解配置：使用Component、Repository、Service、Controller进行注解并区分不同层。通过@Autowired的required（为false表示未找到bean跳过，默认true）和@Qualifier找到名字匹配的bean，普通属性使用value进行注入。（@Resource包含了autowired和qualifier两者功能，更加常见）

作用域：@Scope(value = "singleton/prototype")

生命周期：@PostConstruct构造后执行，@PreDestroy context销毁前执行，singleton才有效

C、xml、注解整合开发，一般开发使用xml管理bean（结构清晰），使用注解进行自动注入（方便快捷），component scan自动开启了annotation-config，如果手动导入包，需要手动开启annotation-config，包扫描包括（service、controller、repository、component等），注解（依赖注入、生命周期中的初始化和销毁方法）扫描需要单独开启annotation-config

**AOP-spring:**

1、传统的业务增强方法使用的是纵向继承（通过父类或者抽象类规定需实现的方法），而aop使用横向抽取机制（底层为代理），自动的扩充类或者其中的方法

2、aop实际上底层实现了spring的动态加载机制（基于JDK的动态proxy代理机制，这样被代理类必须是实现接口），注意动态代理返回的代理类Proxy$0实际上会被转化为被代理类的基类。

3、如果目标类实现了接口，则spring使用底层的JDK动态代理，否则使用CGLIB动态代理。

4、编码过程：

  a、将目标类和切面配置在bean容器中，配置代理类，property应当涵盖目标类（及其基类），切面类

  b、测试中获取代理类，执行方法.

5、如果配置带有切点的切面，则需要对切面进行二次配置，通过正则表达进行方法的匹配。

6、使用传统的proxy代理方式，需要针对每一个被代理类，写一个ProxyFactoryBean，会导致难以维护，后面又出现了基于类名称的自动代理（无法进行方法匹配）和基于正则表达式的自动代理

**AOP-AspectJ:**

1、基于AspectJ的aop编程更加简洁方便，因此spring将AspectJ这个aop框架整合了进来，且采用的编译器织入（静态加载）

2、需要在applicationContext.xml中开启<aop:aspectj-autoproxy/>已启动aop的注解开发方式

3、配置步骤：在bean容器中配置目标类和通知类，配置自动代理，执行目标类方法。

4、前置通知@Before，可以通过配置JoinPoint获取切点的相关信息。

5、通过returning获取返回值

6、通知的类型：Before、AfterReturning、Around、AfterThrowing、After

7、为了将切面和切点分离，方便统一管理，可以单独定制切点方法@Pointcut，并在通知方法上引用。

8、还有xml配置方式，切面类无需使用@Aspect，在bean中配置好dao和advice后，再

```xml
<aop:config>
    <aop:pointcut id="pointcut1" expression="execution(* com.Dao.*(..))"></aop:pointcut>
    <aop:aspect ref="advice">
        <aop:before method="before" pointcut-ref="pointcut1"></aop:before>
    </aop:aspect>
</aop:config>
```

**JDBC-Template：**

1、需要mysql-connector-java、spring-jdbc以及spring-tx（事务处理）包

2、通过配置如下来获得数据库连接池jdbctemplate

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://localhost:3306/selection_course?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false"></property>
    <property name="username" value="root"></property>
    <property name="password" value="54321abc76"></property>
</bean>
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"></property>
</bean>
```

3、execute为可以执行任何语句的方法，但通用性大，导致效率低。

**事务管理：**

1、数据库一套动作的全部执行（成功/回滚），从一种状态转换到另一种状态。遵循ACID原则

2、用途：控制JDBC中数据增删改查。

3、事务属性设置：读取类型（幻读、不可重复读，脏读），隔离级别，传播行为，是否只读，超时设置

4、事务状态，用于获取当前事物的状态，控制事务管理器的运行。

5、编程式事务管理（通过编程实现对事务的管理）：获取事务管理器，对管理器设置属性，设置获取状态，通过管理器获取执行模板，针对业务进行事务提交或回滚。

6、声明式事务管理（基于AOP）：更加关注业务处理，包括tx拦截器(配置)、全注释(注解)

**Spring-mvc：**

1、spring mvc是spring（一站式）的后续延申，因为mvc很好的提供了三大层次的开发，因此spring将springmvc整合了进来，利用ioc、aop等特性，快速开发。

2、核心组件：dispatcherservlet（前置控制器，用于分发不同请求）、handler（处理器，controller）、handlermapping（controller映射器）、handlerInterceptor（拦截器）、handlerexecutionchain（拦截器执行链）、handlerAdapter（处理器适配器）、modelandview（装载试图和模型数据）、viewresolver（视图解析器）

3、配置流程：

在web.xml中配置dispatcherservlet的核心配置文件，通过加载web容器进而加载mvc的配置容器

在springmvc.xml配置handlermapping，将请求路径映射到具体执行的handler

编写controller，最终返回ModelAndView

配置视图解析器viewResolver，通过返回的ModelAndView解析出具体的页面。

4、基于注解的方式，不必编写handlermapping，直接全局扫描controller类，在controller配置@Controller，编写控制器代码和@RequestMapping进行url映射。

5、在controller函数中直接填写传入参数bean，可以直接将请求发过来的参数封装成一个bean对象。除了可以封装对象外，一些基本的数据类型也可以进行参数的自动绑定。

注：如果参数作为路径传入，那么需要在视图函数中对应参数打上@PathVariable

6、基础类型绑定（int、String）直接填写参数，参数名字须相同，对象类型，参数名须与对象属性名相同，list、map和set，须要定制一个list、map和set的包装类，否则无法进行参数绑定，并且也要有set方法，json格式，分为将json格式数据封装为bean，和将bean对象转换为json返回给前端，注意需要导入包“jackson-databind”，和在mvc中进行配置

```xml
<mvc:annotation-driven>
    <mvc:message-converters>
 <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"></bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

**RESTful：**

1、表述性状态转移，资源和url的表述对应关系。

2、统一了资源访问的接口，更加简洁，易懂。

3、put、post、delete、put对应增删改查四个访问接口

4、html的form表单无法直接发送put请求，需要在post表单提交中设置

`<input type="hidden" name="_method" value="put">`

并且在web.xml配置

```xml
<filter>
    <filter-name>hiddenmethod</filter-name>
    <filter-class>
org.springframework.web.filter.HiddenHttpMethodFilter
    </filter-class>
</filter>
<filter-mapping>
    <filter-name>hiddenmethod</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

这样就可以在视图函数中打上@Putmapping接受put的请求了。注意前端只有post可以hidden put和delete请求。

 **过滤器和拦截器区别：**

1、过滤器是servlet规范，处在servlet容器中，拦截器是spring规范，处于spring容器，后者可以通过IOC获取service对象。

2、过滤器位处最外层，是对request请求的过滤，可以实现一些诸如获取请求域和响应域、调用过滤链，重定向、转发等；而拦截器仅包裹在controller外层，是对请求方法的拦截，除了实现以上功能外，还可以获取目标方法Object，返回的异常等信息，使用起来更加灵活多变。拦截器基于AOP动态代理，过滤器使用Filter，过滤request对象。

**其他：**

1、使用idea创建archetype骨架时由于默认是从远程仓库提取资源，因此很慢，使用-darchetypeCatalog=internal参数从本地读取。

2、通过context:property-placeholder locationg=“”指定读取外部配置文件

3、事务,原子性->一致性，隔离性，持久性，事务有三大组件，manager、definition、status，一般事务通过AOP原理实现，具体可通过注解或者tx配置式实现，划分粒度宜为service层，一些复杂的业务操作设置对应的隔离级别和传播行为后，即可确保事务的正确执行。具体的，如果业务中出现了异常，则数据库会执行回滚操作，以保证原子性和一致性。