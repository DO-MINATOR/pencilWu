### 简介

Spring是一个大生态，覆盖了：

- web开发
- 数据访问
- 安全控制
- 分布式
- 消息服务
- 移动开发
- 批处理

![image.png](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/1602641710418-5123a24a-60df-4e26-8c23-1d93b8d998d9.png)

**优点**

- 创建独立的spring应用，方便后续部署微服务，分布式
- 内嵌web服务器，jar可直接运行web
- 依赖spring-boot-start-***，方便构建
- 可通过yaml迅速配置，且三方包会自动寻找配置

**微服务架构风格**

- 一个大型应用拆分为一组小型服务
- 每个服务运行在自己的容器中，独立部署和升级
- 使用http轻量级交互方式
- 全自动话化配置，虚拟化最大效率利用集群资源
- 去中心化，服务自治，可以使用不同语言开发小应用。

**分布式架构风格**

![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/1599562347965-a617a866-4270-44e9-9c5b-ced552683eda.png)

- 远程调用
- 服务发现
- 负载均衡
- 服务容错
- 配置管理
- 服务监控
- 链路追踪
- 日志管理
- 任务调度

解决方案：springboot+springcloud

**云原生需要解决的问题：**

- 服务自愈
- 弹性伸缩
- 服务隔离
- 自动化部署
- 灰度发布
- 流量治理

### 入门

引入依赖

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>//管理了第三方依赖的版本，parent以pom方式打包
    </parent>


    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>//基础包
        </dependency>

    </dependencies>
```

启动程序

```java
/**
 * 主程序类
 * @SpringBootApplication：这是一个SpringBoot应用
 */
@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class,args);
    }
}
```

业务

```java
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String handle01(){
        return "Hello, Spring Boot 2!";
    }
}
```

配置application.properties

```properties
server.port=8888
```

简化部署

```xml
 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>//package打包时可将lib一并导入jar中
            </plugin>
        </plugins>
    </build>
```

#### 项目结构

![image.png](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/1602921758313-5099fe18-4c7b-4417-bf6f-2f40b9028296.png)

#### 配置类configuration

```java
@Configuration(proxyBeanMethods = true) //告诉SpringBoot这是一个配置类 == 配置文件
public class MyConfig {

    /**
     * Full(true):外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
     * Lite(false):不考虑容器中对象，每次创建新对象
     * @return
     */
    @Bean //给容器中添加组件。以方法名作为组件的id。返回类型就是组件类型。返回的值，就是组件在容器中的实例
    public User user01(){
        User zhangsan = new User("zhangsan", 18);
        //user组件依赖了Pet组件
        zhangsan.setPet(tomcatPet());
        return zhangsan;
    }

    @Bean("tom")
    public Pet tomcatPet(){
        return new Pet("tomcat");
    }
}
```

@Bean、@Component、@Controller、@Service、@Repository也可单独加在各包下的类上

#### @Import，配置类中注入

```java
 /* 4、@Import({User.class, DBHelper.class})
 *      给容器中自动创建出这两个类型的组件、默认组件的名字就是全类名
 */
@Import({User.class, DBHelper.class})
@Configuration(proxyBeanMethods = false) //告诉SpringBoot这是一个配置类 == 配置文件
public class MyConfig {
}
```

#### @Conditional条件注入

```java
@Configuration(proxyBeanMethods = false) //告诉SpringBoot这是一个配置类 == 配置文件
@ConditionalOnMissingBean(name = "tom")
public class MyConfig {
    /**
     *只有缺少tom类注入，该配置类及其@Bean声明的类才会被注入。
     */
    @Bean
    public User user01(){
        User zhangsan = new User("zhangsan", 18);
        //user组件依赖了Pet组件
        zhangsan.setPet(tomcatPet());
        return zhangsan;
    }

    @Bean("tom22")
    public Pet tomcatPet(){
        return new Pet("tomcat");
    }
}
```

#### @ConfigurationProperties属性注入

```java
/**
 * 只有在容器中的组件，才会拥有SpringBoot提供的强大功能
 */
@Component
@ConfigurationProperties(prefix = "mycar")//在application.properties(yml)中以mycar开头的配置项的value会被自动导入
public class Car {

    private String brand;
    private Integer price;

//...省略了bean的方法
}
```

### 配置文件yaml

代替原来的properties配置文件

- 字面量：单个的、不可再分的值。date、boolean、string、number、null

```yaml
k: v
```

- 对象：键值对的集合。map、hash、object 

```yaml
行内写法：  k: {k1:v1,k2:v2,k3:v3}
#或
k: 
  k1: v1
  k2: v2
  k3: v3
```

- 数组：一组按次序排列的值。array、list、queue、set

```yaml
行内写法：  k: [v1,v2,v3]
#或者
k:
 - v1
 - v2
 - v3
```

示例


```java
    private String userName;
    private Boolean boss;
    private Date birth;
    private Integer age;
    private Pet pet;
    private String[] interests;
    private List<String> animal;
    private Map<String, Object> score;
    private Set<Double> salarys;
    private Map<String, List<Pet>> allPets;
```

```yaml
# yaml表示以上对象
person:
  userName: zhangsan
  boss: false
  birth: 2019/12/12 20:12:33
  age: 18
  pet: 
    name: tomcat
    weight: 23.4
  interests: [篮球,游泳]
  animal: 
    - jerry
    - mario
  score:
    english: 
      first: 30
      second: 40
      third: 50
    math: [131,140,148]
    chinese: {first: 128,second: 136}
  salarys: [3999,4999.98,5999.99]
  allPets:
    sick:
      - {name: tom}
      - {name: jerry,weight: 47}
    health: [{name: mario,weight: 47}]
```

配置文件添加自定义属性提示

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>


 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>//打包时排除该插件
                    <excludes>
                        <exclude>
                            <groupId>org.springframework.boot</groupId>
                            <artifactId>spring-boot-configuration-processor</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### Web开发

#### 静态资源

访问目录放在/resource/static下

原理： 静态映射/**。

请求进来，先去找Controller看能不能处理。不能处理的所有请求又都交给静态资源处理器。静态资源也找不到则响应404页面

#### webjar

将前端的静态资源如min.js存放在jar中，需引入依赖，如

```xml
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.5.1</version>
        </dependency>
```

`http://localhost:8080/webjars/jquery/3.5.1/jquery.js`后面地址按照依赖路径填写

#### 欢迎页

index.html放在static下作为根目录访问页面

favicon同理

### 请求参数处理

#### rest风格

以前：/getUser   获取用户     /deleteUser 删除用户    /editUser  修改用户       /saveUser 保存用户
现在： /user    GET-获取用户    DELETE-删除用户     PUT-修改用户      POST-保存用户

springboot中开启HiddenHttpMethodFilter，前端提交采用post加_method=PUT/DELETE

#### 参数传递

@PathVariable、@RequestHeader、@ModelAttribute、@RequestParam、@MatrixVariable、@CookieValue、@RequestBody

```java
@RestController
public class ParameterTestController {
    //  car/2/owner/zhangsan
    @GetMapping("/car/{id}/owner/{username}")
    public Map<String,Object> getCar(@PathVariable("id") Integer id,
                                     @PathVariable("username") String name,
                                     @PathVariable Map<String,String> pv){
    }
    
    @PostMapping("/save")
    public Map postMethod(@RequestBody String content){
        Map<String,Object> map = new HashMap<>();
        map.put("content",content);
        return map;
    }
}
```

#### servlet API

WebRequest、ServletRequest、MultipartRequest、 HttpSession、javax.servlet.http.PushBuilder、Principal、InputStream、Reader、HttpMethod、Locale、TimeZone、ZoneId可直接传入函数中作为web内置对象。

**Map**、**Model（map、model里面的数据会被放在request的请求域  request.setAttribute）、**Errors/BindingResult、**RedirectAttributes（ 重定向携带数据）**、**ServletResponse（response）**、SessionStatus、UriComponentsBuilder、ServletUriComponentsBuilder。这些是将数据传递到前端。

### 视图模板解析

springboot不支持jsp，需要引入第三方模板引擎，thymeleaf

#### 引入依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

基本配置

```yaml
spring:
  thymeleaf:
    prefix: classpath:/templates
    suffix: .html
```

| 表达式名字 | 语法   | 用途                               |
| ---------- | ------ | ---------------------------------- |
| 变量取值   | ${...} | 获取请求域、session域、对象等值    |
| 选择变量   | *{...} | 获取上下文对象值                   |
| 消息       | #{...} | 获取国际化等值                     |
| 链接       | @{...} | 生成链接                           |
| 片段表达式 | ~{...} | jsp:include 作用，引入公共页面片段 |

#### 设值属性

```html
<input type="submit" value="Subscribe!" th:value="#{subscribe.submit}"/>
<form action="subscribe.html" th:action="@{/subscribe}">
```

#### 迭代

```html
<tr th:each="prod : ${prods}">
        <td th:text="${prod.name}">Onions</td>
        <td th:text="${prod.price}">2.41</td>
        <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
</tr>
```

#### 条件

```html
<a href="comments.html"
th:href="@{/product/comments(prodId=${prod.id})}"
th:if="${not #lists.isEmpty(prod.comments)}">view</a>
```

### 拦截器

创建登录检查拦截器

```java
/**
 * 登录检查
 */
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {
    /**
     * 目标方法执行之前
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        log.info("preHandle拦截的请求路径是{}",requestURI);

        //登录检查逻辑
        HttpSession session = request.getSession();

        Object loginUser = session.getAttribute("loginUser");

        if(loginUser != null){
            //放行
            return true;
        }

        //拦截住。未登录。跳转到登录页
        request.setAttribute("msg","请先登录");
//        re.sendRedirect("/");
        request.getRequestDispatcher("/").forward(request,response);
        return false;
    }

    /**
     * 目标方法执行完成以后
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle执行{}",modelAndView);
    }

    /**
     * 页面渲染以后
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("afterCompletion执行异常{}",ex);
    }
}
```

配置拦截器

```java
@Configuration
public class AdminWebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")  //所有请求都被拦截包括静态资源
                .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**","/js/**"); //放行的请求
    }
}
```

### 错误处理

配置error/下的4xx，5xx页面会被自动解析

![image.png](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/1606024592756-d4ab8a6b-ec37-426b-8b39-010463603d57.png)