### 导包

**Spring**

aop核心、ioc核心、springjdbc核心

**SpringMvc**

mvc核心、文件上传下载、jstl-jsp标签库、校验码生成、json、日志包

**MyBatis**

核心包、log4j日志包、和spring的整合包spring-mybatis

### 常见整合问题

1. 是否需要进行Spring整合SpringMvc?

   需要，通常Spring整合了数据源、事务、service、dao层，而SpringMvc负责整合controller、视图、文件解析器等。

2. 必须在web.xml中配置启动spring的contextloaderlistner吗？

   不是必须的，也可以在SpringMvc中加载其他spring配置文件，但这样就仅有一个spring容器。

### 整合步骤

web.xml

```xml
<!-- 配置启动 Spring IOC 容器的 Listener -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring.xml</param-value>
</context-param>
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">
 
    <!-- 设置扫描组件的包 -->
    <context:component-scan base-package="com.atguigu.springmvc">
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>
    <!-- 配置数据源, 整合其他框架, 事务等. -->
</beans>
```

springmvc.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:mvc="http://www.springframework.org/schema/mvc"
xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">
 
    <!-- 设置扫描组件的包 -->
    <context:component-scan base-package="com.atguigu.springmvc" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- 配置视图解析器 -->
    <bean id="internalResourceViewResolver"
       class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    <!--静态资源解析器-->
    <mvc:default-servlet-handler/> 
    <mvc:annotation-driven/> 
</beans>
```

### spring和springmvc的关系

![image-20210506093755597](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210506093755597.png)

SpringMVC的容器中的 bean 可以来引用 Spring容器中的 bean。反之则不行

### 整合MyBatis

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.dirver}"></property>
    <property name="url" value="${jdbc.url}"></property>
    <property name="username" value="${jdbc.password}"></property>
    <property name="password" value="${jdbc.password}"></property>
</bean><!--datasource数据源-->

<bean class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="configLocation" value="classpath:mybatis-config.xml"></property>
    <property name="dataSource" value="dataSource"></property>
    <property name="mapperLocations" value="classpath:mapper/*.xml"></property>
</bean><!--配置mybatis的sqlsession-->

<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.wsp.dao"></property>
</bean><!--将MyBatis自动生成的dao实现类添加到IOC容器中，位于mybatis-spring包下-->

<bean id="jdbctransactionmanager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean><!--配置事务控制-->
```

