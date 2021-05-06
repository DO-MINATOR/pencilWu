### 常见整合问题

1. 是否需要进行Spring整合SpringMvc?

   需要，通常Spring整合了数据源、事务、service、dao层，而SpringMvc负责整合controller、视图、文件解析器等。

2. 必须在web.xml中配置启动spring的contextloaderlistner吗？

   不是必须的，也可以在SpringMvc中加载其他spring配置文件，这样就仅有一个spring容器。

### 整合步骤

#### 配置监听器

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
		<context:exclude-filter type="annotation"
expression="org.springframework.stereotype.Controller"/>
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
    <mvc:default-servlet-handler/> 
    <mvc:annotation-driven/> 
</beans>
```

### spring和springmvc的关系

![image-20210506093755597](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210506093755597.png)

SpringMVC的容器中的 bean 可以来引用 Spring容器中的 bean。反之则不行