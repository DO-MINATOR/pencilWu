1、以往的spring开发流程如下：

![image-20200530102531455](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200530102531455.png)

通过spring boot的开发流程简化如下：

![image-20200530102537384](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200530102537384.png)

2、核心特性

  简化开发流程，专注于业务

打包为独立运行的jar包（内部封装tomcat组件），无需再将war发布至tomcat

组件的自动发现与导入依赖，当版本冲突时，可手动导入

提供web应用的运行监控管理

支持大数据spring data、spring cloud优秀框架

3、启动流程

![image-20200530102548728](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200530102601218.png)

其中配置文件默认为空，可以自定义常用配置

启动器同必须以项目名并以Application结尾，入口类@SpringBootApplication修饰，main方法SpringApplication.run(BoottestApplication.class, args);来启动

配置项中的logging.level.root级别如下：

debug<info<warn<error<fatal

4、除了自带的properties配置文件外，也可以使用yml。

![image-20200530102555477](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200530102555477.png)

5、可提供不同环境的配置文件

![image-20200530102601218](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200530102548728.png)

6、可通过maven打包jar，java -jar运行，并可以在当前目录添加

application-{dev}.yml以覆盖jar中配置。

7、spring-boot jdbc配置

```properties
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 54321abc76
    url: jdbc:mysql://localhost:3306/scala?useSSL=false&serverTimezone=GMT
  jpa:
    hibernate:
      ddl-auto: update
    database: mysql
```

8、实体class创建配置

```java
@Entity
@Table
public class MetaDatabase {

    @Id
    @GeneratedValue
    private Integer id;
    private String name;
    private String location;
```

9、repository接口创建

```java
public interface MetaDatabaseRepository extends CrudRepository<MetaDatabase, Integer> {}
```

10、controller中

注解@RestController相当于在@Controller+@ResponseBody此时return语句直接返回前端，可自动识别json格式。

11、好的做法是将restcontroller中返回的json数据进一步封装，使得不同controller返回数据具有统一的格式。

12、scala创建Entity时，使用@BeanProperty等效getter、setter

13、scala的泛型写在[]中

14、scala注入

```java
class MetaTableService @Autowired()(metaTableRepository: MetaTableRepository)
```