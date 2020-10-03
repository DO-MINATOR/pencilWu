1、三者区别

JVM内存结构：JAVA运行时的区域，方法区、堆、栈、程序计数器

  JAVA内存模型JMM：和JAVA并发相关

  Java对象模型：java对象在内存中的表示

2、由于早期C语言没有内存模型，因此指令的执行是基于cpu的，不同的cpu执行同一段代码结果可能是不同的，后来JAVA语言使用了内存，即指令的执行通过内存和cpu共同完成，为了保证同一段代码在不同机器上运行结果一致，就需要jre遵守这样一套规范，即JMM内存模型。

3、JMM主要和并发多线程相关，比如volatile、synchronized等关键字，以及一些工具类（如Lock、内存栅栏CyclicBarrier），底层都是通过JMM实现的。

4、JMM主要包含了：重排序、可见性、原子性

5、重排序：实际执行的顺序有可能与代码编写顺序不同

![image-20200527161623500](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527161623500.png)

重排序3种情况：编译器优化、CPU指令重排序、内存“重排序”（线程A修改的数据没来得及从cache刷回内存中，B线程读取内存导致看似“重排序”）

6、可见性：不同线程可能无法看到对方某一对象真实的值

![image-20200527161650809](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527161628861.png)

  修正可见性的方式为给变量加上关键字volatile，这样每个线程在读取该变量时都会强制缓存中的该变量进行一到flush，然后load进行真实值的操作。

![image-20200527161628861](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527161634597.png)

![image-20200527161657855](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527161636901.png)

  可见性的根本原因是不同cpu核优先读取自己的cache，又可能读取到过期数据，需注意，可见性问题发生原因需要多核+分别使用自己的cache，如果两个线程使用同一个核，则不存在可见性问题。修正其错误的volatile关键字就是JMM规范所提供的，目的就是保证多线程执行与编译器、硬件无关。

![image-20200527161634597](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527161644017.png)

  java在处理可见性问题，提供的JMM规范实际上将cpu缓存和内存抽象成了本地内存和主存，这样即消除了可见性问题，同时又保证了java程序翻译成cpu指令后运用到了cache的快捷性。

7、happens-before原则，A线程先于B线程执行，如果保证c可见性，则A的操作结果对于B是一定可感知的，这就是happens-before原则。

  满足hb原则的情况有：

  单线程、synchronized\lock锁操作、volatile变量、线程start()、线程join、interrupt中断位、工具类(AtomicInteger、CountDownLatch、CyclicBarrier等)

  hb原则还有“就近效应”，即volatile、synchronized前的代码都具有了可见性

8、volatile本身就是一种同步机制。相比于锁同步机制，虽然无法带来线程级别的同步，但可以带来代码级别的同步操作，做不到锁那样的原子保护。因此比锁更加轻量级。

适用于volatile的场合：

在没有synchronized的情况下，如果变量本身的操作就是原子的，那么可以使用volatile来代替synchronized（减少加解锁的开销）；如果使用了synchronized，就没必要再使用volatile了，因为其已经包括了hb操作。

充当触发器，保证可见性。

![image-20200527161636901](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527161650809.png)

  volatile可以禁止指令重排序

**注：synchronized保证可见性只能是，加锁线程读取解锁线程前修改的数据。CountDownLatch同理。**

  java原子操作有：除了（double、long）之外的变量赋值、对象引用、java.Concurrent.Atomic.*所有类

  **注：目前所有商用虚拟机的double、long赋值操作已经保证了原子性**

9、单例模式之双重检查模式

内层的if判断保证实例化一个对象，外层的if判断可以初始化完成后不必再进入synchronized，减少性能损耗。

![image-20200527161644017](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527161657855.png)

  sync中线程A的instance在new步骤为：

​    ①分配新内存

​    ②构造函数执行

​    ③引用赋值

  如果发生了指令重排序，如：①③②，此时线程B（在③后,②之前）第一次判断时，运气好，没发生可见性问题，就直接返回了一个不可用的对象。

  因此，instance须加关键字volatile，主要目的不是为了可见性，而是重排序。

官方推荐使用enum来创建单例，编码简单，饿汉式加载（enum类一旦调用，内部对象全部实例化），JVM加载类的方式保证了线程安全。