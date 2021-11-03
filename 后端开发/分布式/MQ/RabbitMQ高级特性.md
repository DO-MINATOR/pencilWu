### 消息可靠性投递

基础部分介绍了发送方的消息确认机制，分别是confirm和return，前者是发送到exchange成功返回ack，不成功返回nack，后者是发送到exchange但没有queue接收则返回return。

spring中通过配置添加

`publisher-confirms="true`和`publisher-returns="true`来开启confirm和return确认机制。

`rabbitTemplate.setConfirmCalback()`和`rabbitTemplate.setReturnCallback()`设置回调函数。

### 消费端消息确认

消费端收到消息后的确认方式。

- 自动确认：acknowledge="none"
- 手动确认：acknowledge="manual"

手动确认需调用channel.basicAck()，不确认调用channel.basicNack().

### 消费端限流

通过设置消费端每次的预取数量prefetch，实现削峰限流，同时需要确认模式设置为手动。

### TTL

Time To Live，消息的存活时间，如果超过此时间还没有被消费者取出，则将移除消息。可针对队列和消息分别设置过期时间。

- 设置队列过期时间使用参数：x-message-ttl，单位：ms(毫秒)，会对整   个队列消息统一过期。
- 设置消息过期时间使用参数：expiration。单位：ms(毫秒)，当该消息在队列头部时（消费时），会单独判断这一消息是否过期。

### 死信队列

DLX（Dead Letter Exchange），当queue中的消息成为dead message后，可以被重新发送到另一个exchange中。

![image-20211102162235972](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20211102162235972.png)

成为死信的三种情况。

- queue的ttl到期或者message的expire-time到期，都未被消费者读取。
- queue的消息长度达到最大值，其余消息直接被当作dead message。
- 消费者拒绝接收消息，且设置了requeue=false，即不重新入队，而是将消息丢弃，此时直接被当作dead message处理。

注意，consumer1不监听消息，consumer2监听消息，queue1设置过期时间，绑定输出队列DLX，其余exhange和queue的绑定方式与正常工作模式相同，这样时间到时，消息自动被发送到DLX中，consumer2将接收定时消息。

### 延迟队列

RabbitMQ中并未直接提供延迟队列，而是通过TTL+死信队列的方式实现了延时队列。