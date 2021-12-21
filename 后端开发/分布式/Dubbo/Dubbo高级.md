### 负载均衡

消费方进行RPC时，通过一定策略选择集群中不同机器作为服务提供方。

三种策略：

- Random LoadBalance：随机选取。
- RoundRobin LoadBalance：轮询策略。
- LeastActive LoadBalance：最少活跃调用策略，根据消费方调用不同提供方的次数，每次选择调用次数最少的一方，如果相同，则随机选取。每次调用+1，获得结果后次数-1，这样才能反映一段时间内的调用情况。
- ConsistentHash LoadBalance：一致性哈希，相同参数的请求总是发往相同服务提供者，满足幂等性。

### 服务超时

消费者调用一个服务，分为三步：

1. 消费者发送请求（网络传输）
2. 服务端执行服务
3. 服务端返回响应（网络传输）

如果在服务端和消费端只在其中一方配置了timeout，那么没有歧义，表示消费端调用服务的超时时间，消费端如果超过时间还没有收到响应结果，则消费端会抛超时异常**，**但**，**服务端不会抛异常，服务端在执行服务后，会检查执行该服务的时间，如果超过timeout，则会打印一个超时日志**。服务会正常的执行完。

如果在服务端和消费端各配了一个timeout，那就比较复杂了，假设

1. 服务执行为5s
2. 消费端timeout=3s
3. 服务端timeout=6s

### 集群容错

在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。

![image-20211212174844694](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211212174844694.png)

- 这里的 `Invoker` 是 `Provider` 的一个可调用 `Service` 的抽象，`Invoker` 封装了 `Provider` 地址及 `Service` 接口信息
- `Directory` 代表多个 `Invoker`，可以把它看成 `List<Invoker>` ，但与 `List` 不同的是，它的值可能是动态变化的，比如注册中心推送变更
- `Cluster` 将 `Directory` 中的多个 `Invoker` 伪装成一个 `Invoker`，对上层透明，伪装过程包含了容错逻辑，调用失败后，重试另一个
- `Router` 负责从多个 `Invoker` 中按路由规则选出子集，比如读写分离，应用隔离等
- `LoadBalance` 负责从多个 `Invoker` 中选出具体的一个用于本次调用，选的过程包含了负载均衡算法，调用失败后，需要重选

### 服务降级

服务容错是调用失败后，在集群内部重新选举一个本次调用的服务提供者，而降级是单次调用失败后，本服务提供方所采取的后续逻辑，通常有如下配置方案：

- `mock=force:return+null` 表示消费方对该服务的方法调用都直接返回 null 值，不发起远程调用。用来屏蔽不重要服务不可用时对调用方的影响。
- 还可以改为 `mock=fail:return+null` 表示消费方对该服务的方法调用在失败后，再返回 null 值，不抛异常。用来容忍不重要服务不稳定时对调用方的影响。

### 本地伪装

本地伪装就是mock，是服务降级的一种实现方案。

### 本地存根

在执行Dubbo的代理方法之前，执行一端预先定义好的客户端方法，如下是一段stub代码：

```java
public class BarServiceStub implements BarService {
    private final BarService barService;
    
    // 构造函数传入真正的远程代理对象
    public BarServiceStub(BarService barService){
        this.barService = barService;
    }
 
    public String sayHello(String name) {
        // 此代码在客户端执行, 你可以在客户端做ThreadLocal本地缓存，或预先验证参数是否合法，等等
        try {
            return barService.sayHello(name);//真正执行dubbo代理逻辑
        } catch (Exception e) {
            // 你可以容错，可以做任何AOP拦截事项
            return "容错数据";
        }
    }
}
```