### Tomcat架构图

![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/1568877624631-47768292-3817-4c05-9694-f892e4777838.png)

### 解析server.xml

1. 根据配置文件设置启动参数，包括套接字监听端口，最大连接数量，EndPoint对应的是BIO还是NIO，keep-alive属性，socketTimeOut等。
2. 根据Engine、Host、Context创建容器对象，并组织这些容器间的包含关系。
3. 设置每个容器的PipeLine，主要负责request对象的流转问题。

### 启动应用WebApp(Context)

1. 生成一个WebappLoader
2. 生成WebappClassLoader
3. 将WEB-INF/classes和WEB-INF/lib目录作为该加载器的加载path，后续应用如果需要用到对应类库，就从该路径下加载。
4. 解析web.xml
   1. 解析servlet、filters的请求路径和类class的对应关系，存放到类似于servletMappings的对象中。
   2. 将该Mappings配置到Context容器中。

### 启动Connector

1. 启动EndPoint()
2. 如果是NIO，还要创建Poller线程，每个Poller对应一个selector。
3. 当请求IO触发时，快速解析出Request对象，并找到对应的Engine、Host、Context、Servlet容器，逐层通过PipeLine转发下去。

### 热部署和热加载

两者较为类似，前者执行主体是Host，后者是Context。

监听对应class类库或者servlet是否发生变化，是由Tomcat的后台线程来完成的(`BackgroundProcessor`)，每个容器都有对应的`BackgroundProcessor`。

`BackgroundProcessor`负责任务：

- 该服务项目的心跳
- 如果拥有自己的类加载器，检查是否需要热加载。
- 检查session是否过期。
- 检查是否需要热部署

着重分析**热加载**

在相应Context上配置属性reloadable为true，表示该应用开启热加载，默认false。WEB-INF/classes⽬录下的⽂件发⽣了变化，WEB-INF/lib⽬录下的jar包添加、 删除、修改都会触发热加载。

步骤：

1. 设置当前context不能接受处理请求标志为true。
2. 关闭context
3. 开启context
4. 设置当前context不能接受处理请求标志为false。

在第3步中，新启动一个context后，也会创建新的WebappClassLoader，解析web.xml，主要还是查找servlet、filter并记录映射关系到Mappings中，再配置回给server.xml对应的context容器。

以上这些都做完后，就已经具备处理请求的条件了，只有当请求真正过来后，才会执行相关类库和servlet的加载。

而在第2步中，关闭context，其实就是释放相关资源，清空缓存，销毁webappClassLoader，之后的工作就交给JDK的GC。

虽然停止+重启看似做法粗暴了些，但其实是最有效的，相比停掉整个Tomcat再重启，可以省去很多环节的重启，包括server.xml中容器的创建部署，connector的创建等。

**热部署**触发条件相较来说比较大，如果一个web应用整体发生变化（增删改），或者对应context文件发生变化，就需要进行热部署。而热加载仅针对class变化，下面是一些常见条件：

- /tomcat/webapps/应⽤名.war
- /tomcat/webapps/应⽤名
- /tomcat/webapps/应⽤名/META-INF/context.xml
- /tomcat/conf/Catalina/localhost/应⽤名.xml
- /tomcat/conf/context.xml

### Springboot

由于Springboot内置了tomcat，其实本质上和tomcat启动没有太大区别，都需要创建那些容器间的层级关系，context以及类加载器，最终启动一个connector，监听请求。