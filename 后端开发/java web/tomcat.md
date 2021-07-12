### tomcat请求流程

![image-20210702104617367](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210702104617367.png)

1. 从socket获取客户端发来的数据，如果采取的是http1.1协议，且长连接打开，则多个请求按序发送到同一个socket。
2. connector解析数据，包含请求行、请求头、请求体，先解析请求行和请求头。
3. 解析后的字段封装到request中。
4. 初始化一些参数，如connection的keepalive是否close，以及content-length等。
5. 将请求交给容器处理，Engine->Host->Context->Wrapper。
6. 交予具体业务处理器FilterChain->Servlet。
7. 处理完后调用response进行响应，最终flush缓冲区，将数据发送到socket。
8. 如果有多个flush，会先检查当此响应是否已经发送了响应行、响应头。
9. 检查recvbuf中是否还剩余未处理完的请求体，否则处理掉并获取下一个请求行。

### Tomcat架构图

![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/1568877624631-47768292-3817-4c05-9694-f892e4777838.png)

一种树状的层级管理结构，组件有父节点和子节点，父节点管理多个子节点。不同节点的数据传输通过Pipeline传输。

### Tomcat响应流程

当调用OutputStream.write()时，首先会将数据写入到stream对应的缓冲区中，当缓冲区满，或者手动调用flush方法。此后，先判断是否发送过响应头，没有则先发送，再调用flush。

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