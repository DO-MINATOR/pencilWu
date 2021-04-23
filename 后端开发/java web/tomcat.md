### 简介

包含两大核心组件，Connector和Container，并与编程人员定义的Servlet共同组成Service服务。

- Connector：负责接受Http请求，并分配线程处理这个请求，因此connector必然是多线程的。
- Container：service容器，Engine>Host>Context>Wrapper，servlet程序就在Wrapper中，context提供了基本运行环境，管理servlet实例，创建、初始化、销毁等，还提供了内置对象如request、pagecontext、session、application，printwriter等。

其他组件如：Listener、Filter等共同与两大核心组件提供Service服务。

### Tomcat目录

- bin：tomcat可执行程序
- conf：配置文件
- lib：运行支持jar包
- logs：运行时日志
- temp：运行临时文件
- webapps：部署的web工程
- work：工作时缓存文件，如jsp/servlet翻译后的class文件，session钝化后的文件

### 部署工程

两种方式：

1. 将工程目录存放于webapps目录下

2. 在conf/Catalina/localhost/下创建xml文件。配置如下：

   ```xml
   <Context path="abc" docBase="E:\book">
   ```

   这种方式的工程目录不会受到webapps目录的限制

### IDEA下的web工程

![image-20210329111243475](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210329111243475.png)

**修改工程访问路径**

![image-20210329112042004](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210329112042004.png)

**修改端口号**

**修改启动浏览器**

**使用热部署启动**

![image-20210329112253082](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210329112253082.png)