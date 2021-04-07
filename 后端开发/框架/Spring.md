1、面向接口编程，对外提供一个接口，具体的实现由它的实现类去完善，这样可以对外提供一个统一的大功能入口，内部自己根据业务逻辑细分为不同实现类去实现。

2、IOC控制反转，即无需编程人员关注对象及依赖的创建过程，而是将这类控制权反转给框架，并通过依赖注入(DI)的方式实现了对象的创建和维护。在spring中，所有DI的对象都配置成bean，既可以配置文件实现，又可以注解。

3、DI注入方式，因为spring中，一切皆bean，因此无论dao层还是service层，都应在spring配置文件中配置为bean。注入分为设值注入、构造注入（即对bean中的属性值进行赋值，在通过getbean获取对象时，自动的为当前对象属性值赋值）

**IOC：**

4、Bean的配置项有：

  Id Class Scope Constructor-arguments Properties Autowiring-mode 

lazy-initialization-mode initial/destroy-method

5、Bean的作用域：

  singleton(默认)：单例，值一个Bean容器中只有一份（只new 一个对象）

  prototype：每次请求都会实例化一个对象（自动destroy）

  request：一个http请求创建一个实例，在当前request环境下有效

  session：session范围内有效

6、init-method 和destroy-method方法执行bean装配时的初始化和销毁方法。而不在于是否实例化了一个这样的对象，即在工厂形成和销毁时分别执行各个bean的init/destroy方法。bean在装载至IOC容器中，就会执行构造方法和初始化方法。

7、自动装配，和注入中的property和constructor-arg类似，自动装配配置在xml的beans中的default-autowire属性，及当装配某个bean时，检查该bean下的所有属性，是否有属性和id匹配（byName）；是否有类型匹配（byType）；构造器属性类型匹配（constructor）

8、resource用于bean的读取文件，在实现了ApplicationContextAware方法时，就可以通过applicationContext获取到getResource，进而可以读取任何文件，读取方式分为classpath、file和url。

**IOC注解：**

9、使用注解的方式也可以进行bean的装配，@Component表明该类是一个通用bean，@Scpoe定义作用域；同时需要在配置文件中配置

`<context:component-scan base-package="anno.d"></context:component-scan>`

指明需要扫描的包，特别的@Service表示服务层bean，@Repository表示DAO层bean

10、同样的，装配中的自动注入也有注解的方式实现，方式是属性值打上@Autowired，这种方式无需setter方法。该注解还可用于构造器方法和setter方法。常用@Resource（等效于@Autowired、@Qulifier（required="..."））

11、autowired也可使对集合进行注解，包括list和map，spring会自动的将其类及其子类或者类的实现类统一注入到该属性中。对于list，还可以对类打上@Order()，指定注入的顺序。

12、对于多个实现类或者多个子类的情况，可以使用@Qualifier指定注入哪一个bean。

**AOP:**

14、面向切面编程，通常开发人员注重的是业务流的逻辑功能开发，而不同的业务都会有一些相同的公共操作，比如日志记录，性能统计，事务处理等操作，如果针对每一个业务流都去编写这些实现代码，可想而知，不易维护（代码入侵）。因此，AOP就提供了一个这样的功能，通过动态代理或者类增强进行方法的功能扩充。

15、aop的配置同样可以以配置文件的方式进行配置，

```xml
<bean id="yewu" class="aop.yewu"></bean>
<bean id="aspect" class="aop.aspect"></bean>

<aop:config>
    <aop:aspect id="aop_test" ref="aspect">
        <aop:pointcut id="point_cuttest" expression="execution(*aop.yewu.*())"></aop:pointcut>
        <aop:before method="before" pointcut-ref="point_cuttest"></aop:before>
    </aop:aspect>
</aop:config>
```

完整的配置如上，通过xml方式引入bean，yewu为普通的service，aspect为定义的aop类；在aop：config中配置切面aspect，point-cut为切点（可以有多个），before就是执行切点前的方法。

16、除了before，还有after-returning（切点返回后执行），after-throwing（切点抛出异常后执行）after（无论异常还是正常返回，都会执行），以及环绕通知around和通知的参数匹配

注意：指定的切点中的方法为本类声明的，继承的方法不算。

17、通过aop:declare-parents，可以为指定对象声明一个它实现的类。

**AOP注解：**

18、aop也有注解的方式，对于通知类，需要打上@Aspect和@Component，只打第一个不会被context:component-scan自动装配，同时配置文件写上

<aop:aspectj-autoproxy></aop:aspectj-autoproxy>

19、切面类定义

切入点：

```java
@Pointcut("execution(* aop.yewu.test1())")
public void poincut() {
}
```

切入方法：

```java
@Before("poincut()")
public void before() {
    System.out.println("before");
}
```

after、afterreturing、around、agter-throwing实现方式一样

20、给切面传入参数的方式，即在切点声明args(arg...)，然后在切点的执行方法处括号内引用该参数，需要注意参数名称需相同

**其他：**

1、使用<context:property-placeholder location:"">在spring.xml中引入外部文件，再使用${}指定引入外部文件中的某一个属性值

2、spring框架中有用于整合junit4的jar包，可用于将一个测试类自动的导入spring配置文件，然后通过@Resource引入bean

3、事务管理，spring提供了对数据库业务逻辑的事务管理方案。包括编程式事务管理和声明式事务管理，以保证事务的原子性、一致性、持久性、隔离性。

4、事务的7种传播行为，其中PROPAGATION_REQUIRED比较常用，即如果当前上下文有事务，就直接加入到该事务中，否则创建一个事务。







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

##### 原理

- xml解析
- 工厂模式、单例模式
- 反射

##### 图示过程

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

1. BeanFactory：开发人员一般不使用。对象在使用时才创建
2. ApplicationContext：BeanFactory接口的子接口，功能更强大。加载配置文件随机创建对象

```java
@Test
public void test1(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("Spring.xml");
    User user = applicationContext.getBean("user", User.class);
    user.add();
}
```

#### Bean管理

##### xml

```xml
<bean id="user" class="...">
```

id：获取对象的标识
class：类的全限定名
创建对象默认调用无参构造方法。
DI：依赖注入的具体实现。

```xml
<bean id="service" class="...">
    <property name="name" ref="hello"></property>
    <property name="userdao" ref="daoImpl"></property>
</bean>
```

数组、List、Map和Set集合类属性的注入，注意，对于集合类，Spring有它默认的实现类，比如List是LinkedList，Map是LinkedHashMap。

##### 注解

相对xml更加简洁。

1. 引入依赖spring-aop

2. 开启组件扫描

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
   
       <context:component-scan base-package="service"></context:component-scan>
   </beans>
   ```

3. 依赖注入

   - 对象类型注入：@Autowired（根据类型）、@Qualifier（根据名称，需与autowired一起使用）、@Resource（前两者的混合体）普通类型注入：
   - 普通类型注入：@Value

#### 生命周期

1. 调用对象的构造器方法
2. 设置对象依赖
3. 调用bean的初始化方法（需要手动配置）
4. 使用bean
5. 容器关闭时，调用bean的销毁方法（需要手动配置）

#### xml管理

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

#### 原理

- 有接口的情况，JDK动态代理，创建的是接口的实现类作为代理对象
  ![image-20210407211153062](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210407211153062.png)
- 没有接口的情况，CGLIB动态代理，创建的是被代理对象的子类作为代理对象
  ![image-20210407211312227](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210407211312227.png)

