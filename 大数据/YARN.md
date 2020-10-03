1、YARN产生背景

![image-20200519204802597](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519204802597.png)

  JobTracker单点压力大，既要接受client的任务请求，还要向的datanodes分发任务

![image-20200519204805246](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519204809820.png)

  资源利用率不够充分，需要结合YARN多框架统一调度管理。

 2、YARN执行流程

![image-20200519204809820](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519204805246.png)

Client向RM提交作业请求，RM与某一NM提交作业分配，启动对应AppM（集群内监管任务第一负责人），同时AppM反向注册到RM（成为集群内监管任务第二负责人），AppM如果需要其他block资源，还需要向其他NM安排container，所有container内可以运行任意计算框架，container将结果汇总至AppM，最终返回给RM->Client。

3、环境配置

在mapred-site.xml下配置

```xml
<property>
	<name>mapreduce.framework.name</name>
	<value>yarn</value>
</property>
```

在yarn-site.xml下配置

```xml
<property>
    <name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>
</property>
<property>
	<name>yarn.nodemanager.local-dirs</name>
	value>/home/hadoop/app/tmp/nm-local-dir</value>
</property>
```

在sbin目录下./start-yarn.sh启动服务

4、提交本地MR作业到YARN运行

  将MR作业打包，上传jar包和相应数据到linux

  hadoop jar 包名 主类名 args...（输入文件 输出文件） 启动Yarn上的MR作业。