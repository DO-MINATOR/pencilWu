### 简介

MQ全程Message Queue（消息队列），在消息传输过程中保存消息的容器，多用于分布式系统中组件间的通信。

![image-20211101110106311](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211101110106311.png)

图中生产者、消费者都属于客户端，向MQ发送、接受数据。

### 优势

- 应用解耦，相比RPC，客户端只需要与MQ进行交互，收发数据。
- 异步提速，MQ更像是一个异步消息队列，发送方只管发送，无需确认数据是否被正确接受。

![image-20211101110528470](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211101110528470.png)

- 削峰填谷，无论多少请求，发送到消息队列中，消费端按照负载状况取数据。

### 劣势

- 提升了系统整体复杂性，一旦MQ宕机，系统将被阻塞，需要保证MQ的高可用。
- 业务逻辑变得更加复杂，RPC是同步调用，MQ是异步消息队列，如何保证业务逻辑正常执行。

### 架构流程

![image-20211101111102756](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211101111102756.png)

- Virtual host：虚拟机，出于多租户和安全因素考虑，类似于网络空间的namespace，用户可以创建自己的host，并分配权限给其它用户。
- connection：客户端和MQ的TCP连接。
- channel：轻量级连接，避免了重复创建TCP链接造成的大量开销，内部包含channel id用以区分不同channel。
- exchange：分发规则，匹配查询表中的routing key，发送到对应的queue。

### 工作模式

- simple：p生产消息发送给MQ，c从MQ中取数据。
- work queue：相比simple，有多个c，从同一个queue中取数据。适用于任务较重且拥有多个消费端集群，可以配置消费端取数据的方式（轮询、先到先得...）
- publish/subscribe
- routing
- topic
- rpc（远程调用模式，不算MQ）

![image-20211101112034226](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211101112034226.png)

### 执行流程

#### 创建连接

```java
private static ConnectionFactory connectionFactory = new ConnectionFactory();
static {
    connectionFactory.setHost("81.71.140.7");
    connectionFactory.setPort(5672);//5672是RabbitMQ的默认端口号
    connectionFactory.setUsername("bq123");
    connectionFactory.setPassword("bq123");
    connectionFactory.setVirtualHost("/baiqi");
}
public static Connection getConnection(){
    Connection conn = null;
    try {
        conn = connectionFactory.newConnection();
        return conn;
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```

#### 发送数据

```java
//获取TCP长连接
Connection conn = RabbitUtils.getConnection();
//创建通信“通道”，相当于TCP中的虚拟连接
Channel channel = conn.createChannel();
//创建队列,声明并创建一个队列，如果队列已存在，则使用这个队列
//第一个参数：队列名称ID
//第二个参数：是否持久化，false对应不持久化数据，MQ停掉数据就会丢失
//第三个参数：是否队列私有化，false则代表所有消费者都可以访问，true代表只有第一次拥有它的消费者才能一直使用，其他消费者不让访问
//第四个：是否自动删除,false代表连接停掉后不自动删除掉这个队列
//其他额外的参数, null
channel.queueDeclare(RabbitConstant.QUEUE_HELLOWORLD,false, false, false, null);

String message = "hello白起666";
//exchange 交换机，暂时用不到，在后面进行发布订阅时才会用到
//队列名称
//额外的设置属性
//最后一个参数是要传递的消息字节数组
channel.basicPublish("", RabbitConstant.QUEUE_HELLOWORLD, null,message.getBytes());
channel.close();
conn.close();
System.out.println("===发送成功===");
```

#### 接收数据

**simple**

```java
//获取TCP长连接
Connection conn = RabbitUtils.getConnection();
//创建通信“通道”，相当于TCP中的虚拟连接
Channel channel = conn.createChannel();
//创建队列,声明并创建一个队列，如果队列已存在，则使用这个队列
//第一个参数：队列名称ID
//第二个参数：是否持久化，false对应不持久化数据，MQ停掉数据就会丢失
//第三个参数：是否队列私有化，false则代表所有消费者都可以访问，true代表只有第一次拥有它的消费者才能一直使用，其他消费者不让访问
//第四个：是否自动删除,false代表连接停掉后不自动删除掉这个队列
//其他额外的参数, null
channel.queueDeclare(RabbitConstant.QUEUE_HELLOWORLD,false, false, false, null);

//创建一个消息消费者
//第一个参数：队列名
//第二个参数代表是否自动确认收到消息，false代表手动编程来确认消息，这是MQ的推荐做法
//第三个参数要传入DefaultConsumer的实现类
channel.basicConsume(RabbitConstant.QUEUE_HELLOWORLD, false, new Reciver(channel));

class Reciver extends DefaultConsumer {
    private Channel channel;
    //重写构造函数,Channel通道对象需要从外层传入，在handleDelivery中要用到
    public Reciver(Channel channel) {
        super(channel);
        this.channel = channel;
    }

    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        String message = new String(body);
        System.out.println("消费者接收到的消息："+message);
        System.out.println("消息的TagId："+envelope.getDeliveryTag());
        //false只确认签收当前的消息，设置为true的时候则代表签收该消费者所有未签收的消息
        channel.basicAck(envelope.getDeliveryTag(), false);
    }
}
```

**work queue**

```java
Connection connection = RabbitUtils.getConnection();
final Channel channel = connection.createChannel();
channel.queueDeclare(RabbitConstant.QUEUE_SMS, false, false, false, null);
//如果不写basicQos（1），则自动MQ会将所有请求平均发送给所有消费者
//basicQos,MQ不再对消费者一次发送多个请求，而是消费者处理完一个消息后（确认后），在从队列中获取一个新的
channel.basicQos(1);//处理完一个取一个
channel.basicConsume(RabbitConstant.QUEUE_SMS , false , new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        String jsonSMS = new String(body);
        System.out.println("SMSSender1-短信发送成功:" + jsonSMS);
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        channel.basicAck(envelope.getDeliveryTag() , false);
    }
});
```

#### pub&sub/routing/topic收发数据

![image-20211101200022739](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211101200022739.png)

相较于simple/queue模式，多了一个exchange交换机，发送方将数据发送给交换机，（需先设置交换机的工作模式），消费者仍需指定从哪一个队列取数据，同时需要根据工作模式将queue绑定到router上。

这三种不同的模式对应了exchange的三种不同工作方式，分别是fanout（将数据广播到绑定的queue上）、direct（将数据根据key发送到不同queue）、topic（将数据根据key的通配符匹配规则发送到queue）。注意exchange只负责转发数据，如果没有消费者绑定队列到交换机上，那么数据将会丢失。

### 消息确认机制

#### Confirm

- ack：exchange收到数据
- nack：exhange拒收数据，可能是队列已满，限流，io异常等

#### Return

当exhange返回ack后，如果没有对应queue消费数据，则会返回Return信息。

上面两种机制与消费者的ack无关。

### Spring整合

生产端

- 创建spring工程，引入依赖
- 配置yml的mq连接信息
- 定义exchange、queue及其绑定关系
- 注入rabbitTemplate，调用方法，发送消息。

消费端

- 创建spring工程，引入依赖
- 配置yml的mq连接信息
- 定义监听器
- 注入rabbitListener，调用方法，接收消息

