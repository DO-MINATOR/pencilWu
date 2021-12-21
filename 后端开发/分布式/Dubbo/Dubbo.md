### RPC

在分布式计算**，远程过程调用**（英语：Remote Procedure Call，缩写为 RPC）是一个计算机通信协议。该协议允许调用另一个[地址空间的子程序，而程序员就像调用本地程序一样，无需额外地为这个交互作用编程（无需关注细节）。RPC是一种服务器-客户端（Client/Server）模式，经典实现是一个通过**发送请求-接受回应**进行信息交互的系统。

如果涉及的软件采用面向对象编程，那么远程过程调用亦可称作**远程调用**或**远程方法调用**。

相对的，本地方法而言是在同一个JVM进程下的方法调用。

远程调用有两大类实现方式：

- RPC over Http：基于Http协议来传输数据
- PRC over Tcp：基于Tcp协议来传输数据

因此，RPC前一般会协商好传输协议，字段说明。就像网络接口提供接口说明，方法调用提供参数说明。

### Dubbo

Dubbo是阿里开源的一套RPC框架（起初是专门为RPC开发的，后来涉足到微服务很多功能），目前作为Apache的顶级项目。

**基本原理**

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211208120152569.png" alt="image-20211208120152569" style="zoom: 57%;" />

### Dubbo与Spring整合

![image-20211209212334627](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211209212334627.png)

Dubbo主要需要做的就是生产者**服务导出**和消费者**服务引入**。

在生产方，由Dubbo生成serviceBean对象，其属性根据Dubbo配置文件，注入相应的protocol、registries、configcenter、provider、group、version、mock等，另还有一个ref属性指向真正的服务对象serviceImpl，该对象由Spring管理。后续serviceBean对象根据属性信息将服务暴露在注册中心上。

在消费方，同样由Dubbo生成referenceBean对象，该对象有和serviceBean相类似的属性，通过这些属性生成真正的代理对象，代理方法中包含Dubbo的远程调用逻辑（如Dubbo20880访问）

