1、相比离线处理，实时流处理更加注重时效性，因此数据来源通常从消息队列，处理速度也需要更加快速。另外，相比离线处理的进程机制，实时流处理通常常驻与内存中。

2、集群中有很多的servers，首先需要将日志文件从不同机器中读取到hadoop集群中，单纯的cp.shell方式存在无法监控、IO忙、时效性、不能压缩等问题，因此需要Flume框架。

![](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520174459473.png)

3、执行步骤：

  a、编写配置文件

![image-20200520174515335](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520174508656.png)

  b、启动Flume

![image-20200520174527283](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520174515335.png)

  c、对监听的source写入数据

![image-20200520174459473](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520174522447.png)

  d、从sink输出的目标查看数据

![image-20200520174522447](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200520174527283.png)

4、如果要改成exec-memory-logger模式，则

```conf
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /home/hadoop/data/data.log
a1.sources.r1.shell = /bin/sh -c
```

5、常见的作法是将A服务器的log数据通过exec-memory-arvo输出到B服务器（hadoop集群）上的arvo-memoryexec（logger）上。因此需要配置两个flume-conf。注意：应先启动avro-memory-log，否则exec-memory-avro会找不到指定输出的端口（因为这是一个推流）。