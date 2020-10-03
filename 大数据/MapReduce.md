1、MapReduce，适用于海量的数据离线处理，使用编程接口即可完成分布式的数据处理。但不适用于实时的数据流计算。

2、处理流程

![image-20200519203944357](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519203944357.png)

在MapReduce模型中，只注重Mapping和Reducing这两步的操作

![image-20200519203949562](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200519203949562.png)

核心模块为Split、InputFormat、OutputFormat、Partitioner、Combiner

3、统计文件词频

  编写Mapper类

```java
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] words = value.toString().toLowerCase().split("\t");
        for (String word : words) {
            context.write(new Text(word), new IntWritable(1));
        }
    }
}
```

继承的Mapper类的四个泛型依次为输入偏移量offset、字符串、输出key、输出value。覆写的函数中，key为偏移量，value为字符串，context为上下文缓存，即把输出至缓存中。

编写Reducer类

```java
public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int count = 0;
        Iterator<IntWritable> iterator = values.iterator();
        while (iterator.hasNext()) {
            IntWritable value = iterator.next();
            count += value.get();
        }
        context.write(key, new IntWritable(count));
    }
}
```

四个泛型依次为输入key、输入value、输出key、输出value。函数参数中key为输入关键字，values是可迭代的value对象。

编写driver整合类，同样配置configuration中的hdfs路径，文件计数

```java
Job.getInstance(configuration)
```

获取任务启动类（一般为本类），设置mapper、reducer类，combiner类（可选的），设置mapper输出key、输出value类型，reducer的输出key、输出value类型，通过FileInputFormat设置文件输入输出路径，job.waitForCompletion(verbose：true)启动任务，为true表示大作业显示进度，false直接输出结果。

4、设置CombinerClass，在map完成以后，无论key是否相同，entry都是单个输出的，如果不加以聚合，则在shuffle时，将会导致大量的IO和网络传输，因此添加combine聚合操作，所使用的类方法和reduce相同，因此可以复用。

5、更改输出value的类型。自定义类型需要implements Writable，其中write和readFields方法需要按照同一字段顺序进行读写。输出空的类型为NullWritable，使用NullWritable.get()。

6、设置PartitionerClass，在shuffle完成以后，所有的输出结果都将传输到同一个ReduceTask（job默认设置setNumReduceTasks为1），因此编写PartitionerClass可以将entry按照一定规则输出到不同的ReduceTask，自定义PartitionerClass需继承Partitioner，同时job设置与之对应的setNumReduceTasks。