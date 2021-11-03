### 分布式锁

#### 非公平锁

**加锁原理**

由于zookeeper类似redis的单线程执行，且节点只能创建一次，因此先创建的成功获取到锁，后面的会返回失败，并尝试监听该节点的释放，再尝试获取锁。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211026103717168.png" alt="image-20211026103717168" style="zoom:67%;" />

该锁对应的是非公平锁，且性能较差，每次释放锁后，通知事件占用了大量网络IO。

#### 公平锁

借助顺序临时节点，可以有效提升网络性能，同时保持公平性，通过监听前一个节点的释放，来保证公平性。

![image-20211026104206999](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211026104206999.png)

#### 共享锁

之前两种锁在访问量大的场景下不适用，且存在读写不一致、双写不一致问题，因此采用读写锁，实现原理如下：

![image-20211026104505172](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211026104505172.png)

写请求始终监听前一个节点的释放，读请求监听上一个写节点的释放。

### 选举



### 注册中心

在环境较为简单时，服务调用可能是这样的，即单一点对点提供服务。

![image-20211026104718895](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211026104718895.png)

如果随着服务的扩展，User-service可能会进行集群部署，服务调用就可能变成这样。

![image-20211026104816872](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211026104816872.png)

此时通过Nginx反向代理也可以实现负载均衡，但如果服务接口数量继续增长，调用关系变为如下所示：

![image-20211026105048644](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211026105048644.png)

这个时候我们可以借助于Zookeeper的基本特性来实现一个注册中心,什么是注册中心，顾名思义，就是让众多的服务，都在Zookeeper中进行注册，啥是注册，注册就是把自己的一些服务信息，比如IP，端口，还有一些更加具体的服务信息，都写到 Zookeeper节点上， 这样有需要的服务就可以直接从zookeeper上面去拿，怎么拿呢？ 这时我们可以定义统一的名称，比如，User-Service, 那所有的**用户服务**在**启动**的时候，都在User-Service 这个节点下面创建一个子节点（临时节点），这个子节点保持唯一就好，代表了每个服务实例的唯一标识，有依赖**用户服务**的比如**Order-Service** 就可以通过**User-Service** 这个父节点，就能获取所有的User-Service 子节点，并且获取所有的子节点信息（IP，端口等信息），拿到子节点的数据后**Order-Service**可以对其进行缓存，然后实现一个客户端的负载均衡，同时还可以对这个User-Service 目录进行监听， 这样有新的节点加入，或者退出，**Order-Service**都能收到通知，这样**Order-Service**重新获取所有子节点，且进行数据更新。这个用户服务的子节点的类型为临时节点。 第一节课有讲过，Zookeeper中临时节点生命周期是和SESSION绑定的，如果SESSION超时了，对应的节点会被删除，被删除时，Zookeeper 会通知对该节点父节点进行监听的客户端, 这样对应的客户端又可以刷新本地缓存了。当有新服务加入时，同样也会通知对应的客户端，刷新本地缓存，要达到这个目标需要客户端重复的注册对父节点的监听。这样就实现了服务的自动注册和自动退出。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211026105221902.png" alt="image-20211026105221902" style="zoom:80%;" />

Spring Cloud提供了Zookeeper的注册中心实现（Spring Cloud Zookeeper），被调用方自动注册到zookeeper，调用方主动发现节点，并以逻辑节点访问的方式调用服务，并能实现负载均衡。

