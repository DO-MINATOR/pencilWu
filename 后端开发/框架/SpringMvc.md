### 拦截器

1、拦截器和过滤器运行原理相似，回顾过滤器在web.xml中配置的，对于所有请求都会经过指定的过滤器，进行请求和返回的处理。（编码问题处理等）

2、拦截器的配置实在mvc-dispatcher-servlet中（与web.xml过滤器不同），且需要实现HandlerInterceptor接口，并在配置文件中配置路由规则和相应的处理类

3、interceptor有三个方法，分别是prehandle(有返回值，位false时直接阻断请求)posthandle（请求处理完，返回响应之前执行，可通过modelandview修改传入view的属性对象或者修改返回界面字符串）afterCompletion（响应返回后执行），它们都有Object对象，为请求者的请求目标对象，即controller类。

4、多个拦截器协同工作时，按照注册的顺序执行，pre1->pre2->post2->post1->after2->after1

5、过滤器基于servlet容器，过滤范围较大（包括静态资源），拦截器基于spring容器，仅过滤针对controller的请求。两者都可以减少代码复用，便于维护。

### 简介

- Model（模型）： 数据模型，提供要展示的数据，：Value Object（数据Dao） 和 服务层（行为Service），提供数据和业务。

- View（视图）： 负责进行模型的展示，即用户界面

- Controller（控制器）： 调度员，接收用户请求，委托给模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示。


### 特点

- SpringMVC通过一套MVC注解，只需一个@controller即可声明一个控制器。
- 支持REST风格的URL请求
- 采用了松散耦合可拔插组件结构，扩展性和灵活性

### 步骤

导入依赖

```xml
<dependencies>
        <!-- SpringWeb基础包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>4.0.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.0.0.RELEASE</version>
        </dependency>
        		<!--        核心包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.0.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>4.0.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>4.0.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>4.0.0.RELEASE</version>
        </dependency>
       <!--       日志包-->
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.1.3</version>
        </dependency>
        		<!--        注解支持包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>4.0.0.RELEASE</version>
        </dependency>
</dependencies>
```

配置web.xml，注册DispatchServlet（前端控制器，负责请求分发）

```xml
        <servlet>
            <servlet-name>springDispatcherServlet</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>classpath:springmvc-config.xml</param-value><!--导入spring配置文件-->
            </init-param>
            <load-on-startup>1</load-on-startup>
        </servlet>
        <servlet-mapping>
            <servlet-name>springDispatcherServlet</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>
```

配置spring.xml，开启注解扫描，以及视图解析器。

```xml
<!-- 告诉Spring MVC自己映射的请求就自己处理，不能处理的请求直接交给tomcat -->
<mvc:default-servlet-handler />
<!--开启MVC注解驱动模式，保证动态请求和静态请求都能访问-->
<mvc:annotation-driven/>
<context:component-scan base-package="com.wsp.controller"/>
    <!--配置视图解析器，拼接视图名字，找到对应的视图-->
<bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<!--前缀-->
	<property name="prefix" value="/WEB-INF/page/"/>
    <!--后缀-->
    <property name="suffix" value=".jsp"/>
</bean>
```

编写controller，并实现以下注解

1. @Controller：告诉Spring这是一个控制器，交给IOC容器管理
2. @RequestMapping("/hello01")：/ 表示项目地址，当请求项目中的hello01时，返回一个/WEB-INF/page/success.jsp页面给前端

```java
@Controller
public class HelloController {
    @RequestMapping(value="hello01",method=GET)
    public String toSuccess(){
        System.out.println("请求成功页面");
        return "success";
    }
}
```

- value表示路径

- method表示请求方法，GET、POST、PUT、DELETE


### REST风格

一种资源+操作符的路径请求规范，因为HTTP中有四种请求方法（GET/PUT/POST/DELETE）正好对应了增删改查四种方法。相比原来的写法如addbook?id=1、querybook?id=10，使用book/1并分别结合POST、GET查询操作能够更好的表示对资源的请求。

实现步骤：

1. 配置Filter

   ```xml
   <filter>
       <filter-name>HiddenHttpMethodFilter</filter-name>
       <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
   </filter>
   <filter-mapping>
       <filter-name>HiddenHttpMethodFilter</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
   ```

2. 创建form表单，并新增name="_method"项，value为"put/delete"

   ```jsp
   <form action="hello03" method="post">
     <input type="hidden" name="_method" value="delete">
     <input type="submit" name="提交">
   </form>
   ```

高版本Tomcat会出现问题：JSP only permit GET POST or HEAD，在页面上加上异常声明即可

   ```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java"  isErrorPage="true" %>
   ```

###  请求参数获取

如果请求参数和处理方法中参数名一致，则直接使用。

```java
@RequestMapping("/hello05")
public String test03(String username) {
    System.out.println(username);
    return "success";
}
//请求路径：http://localhost:8080/hello05?username=zhangsan，则输出zhangsan
```

如果请求参数和处理方法中参数名不一致，则可通过**@RequestParam**修饰。

```java
@RequestMapping("/hello05")
public String test03(@RequestParam("user")String username) {
    System.out.println(username);
    return "success";
}
//请求路径：http://localhost:8080/hello05?user=zhangsan，则输出zhangsan
```

将请求路径作为参数传入方法中，区别于上面的?和&参数传递

```java
//获取到{id}占位符，占位符可以在任意路径地方写{变量名}
//@PathVariable("id") 获取请求路径哪个占位符的值，请求路径名字不一定是"id"
@RequestMapping(value = "/hello05/{id}")
public String myMethodTest03(@PathVariable("id") String id) {
    System.out.println("路径上占位符"+id);
    return "success"；
}
```
在获取请求参数中，如果参数数量过多，还可以将参数封装为POJO对象，需要form表单和javabean对象。

```jsp
<form action="book" method="post">
	<input type="text" name="bookname"/>
	<input type="text" name="author"/>
	<input type="text" name="price"/>
	<input type="text" name="stock"/>
	<input type="text" name="address.province"/>
	<input type="text" name="address.city"/>
	<input type="submit"/>
</form>
<!--可级联绑定-->
```

```java
public class Book{
	private String bookname;
	private String author;
	private Integer price;
	private Integer stock;
	private Address address;
	...
}
public class Address{
	private String province;
	private String city;
	...
}

@RequestMapping("/book")
public String test03(Book book) {
    System.out.println(book);
    return "success";
}
```

### Cookie获取

```java
@RequestMapping("/hello05")
public String test03(@CookieValue("JSESSIONID")String name) {
     System.out.println(name);
     return "success";
}
```

### 传入原生ServletAPI

在请求方法中直接添加对应参数，如

```java
@RequestMapping("/servletapi")
public String test03(HttpServletRequest request, HttpServletResponse response, HttpSession session) {
     session.setAttribute("name","Alex")
     return "success";
}
```

### 字符集

设置post请求utf-8编码

```xml
<filter>
   <filter-name>encoding</filter-name>
   <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!--解决请求乱码-->
   <init-param>
       <param-name>encoding</param-name>
       <param-value>utf-8</param-value>
   </init-param>
    <!--解决响应乱码-->
   <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
   </init-param>
</filter>
<filter-mapping>
   <filter-name>encoding</filter-name>
   <url-pattern>/*</url-pattern>
</filter-mapping>
```

设置get请求utf-8编码

```xml
<!--在Tomcat的server.xml中的8080处 URLEncoding="UTF-8"-->
```

### 数据传出至View

除了可以通过servlet原生API进行数据绑定，也可以在方法中加入Map、Model、ModelMap对象，这些都会实例化为**BindingAwareModelMap**对象。

```java
@RequestMapping("/Api2")
public String api2(Map<String,Object> map){
	map.put("msg","hello");
	return "map";
} 
@RequestMapping("/Api3")
public String api3(Model model){
	model.addAttribute("msg","hello2");
	return "map";
}

@RequestMapping("/Api4")
public String api4(ModelMap modelMap){
	modelMap.addAttribute("msg","hello3");
	return "map";
}
```

前端页面


```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
pageScope:  		${pageScope.msg}
requestScope :  	${requestScope.msg}
sessionScope:     	${sessionScope.msg}
applicationScope:   ${applicationScope.msg}<!--上述三个对象绑定数据至前端，前端只能通过request域获取数据-->
</body>
</html>
```

为了给session中也保存上该数据，除了可以用原生的API HttpSession外，还可以通过@SessionAttributes，该注解只能表在类上，value指定key，type指定保存的类型，如：

```java
//表示给BindingAwareModelMap中保存key为msg的数据时，在session中也保存一份，如果保存的类型为String，则也会在session中保存。
@SessionAttributes(value = "msg" type="String.class")
@Controller
public class outputController {
    @RequestMapping("/hello01")
    public String test01 (Map<String,Object> map){
        map.put("msg","HelloWorld!");
        return "success";
    }
}
```

