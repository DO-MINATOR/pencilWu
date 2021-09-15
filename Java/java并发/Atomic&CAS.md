### Atomic

底层通过Unsafe提供的无锁原语CAS实现。

例如AtomicInteger，执行逻辑如下

```java
do {
    oldvalue = this.getIntVolatile(var1, var2);//读AtomicInteger的value值
    ///valueOffset---value属性在对象内存当中的偏移量
} while(!this.compareAndSwapInt(AtomicInteger, valueOffset, oldvalue, oldvalue + 1));
```

偏移量用于获取对象中某一属性在该对象中的所属内存区域的偏移地址。

- 基本类：AtomicInteger、AtomicLong、AtomicBoolean；
- 引用类型：AtomicReference、AtomicStampedRerence(防止ABA问题)；
- 数组类型：AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray
- 属性原子修改器（Updater）：AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicReferenceFieldUpdater

注意，原子引用变量调用`compareAndSet()`时，判断的是指向是否相同，而不是equals，AtomicInteger判断的是内容是否相同。

### CAS执行流程

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/Compare And Swap【更多一手资源加微信itit11223344】.jpg" alt="Compare And Swap" style="zoom: 60%;" />

cas的执行是原子的，一次执行只能有一个线程能够完成，且操作的变量由volatile修饰，所以可以保证可见性，第一个线程执行了cas逻辑，第二个线程执行cas时可以立即看见之前修改的变量值。

AtomicInteger常用方法：

- int addAndGet(int delta) ：以原子方式将输入的数值与实例中的值（AtomicInteger里的value）相加，并返回结果
- boolean compareAndSet(int expect, int update) ：如果输入的数值等于预期值，则以原子方式将该值设置为输入的值。
- int getAndIncrement()：以原子方式将当前值加1。
- void lazySet(int newValue)：最终会设置成newValue，使用lazySet设置值后，可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
- int getAndSet(int newValue)：以原子方式设置为newValue的值，并返回旧值。

### CAS执行特点

1. 相比锁，执行粒度更小，提升系统效率

2. 如果竞争高度激烈，则会由于不断重试反而降低了cpu效率，不如使用锁机制来让cpu决定何时调度执行线程。

**CAS是乐观锁、并发容器、原子类、AQS的实现核心**

CAS是CPU原语，默认保证了原子操作，即多个线程只有一个能够执行CAS操作，CAS包含三个参数（memValue，expectValue，newValue），第一个为变量在内存中的值，之后其中一个线程成功修改了该变量，其余变量之后执行都会返回false，可以根据返回的布尔值决定退出线程还是while重试，如果采取重试策略，要注意expectValue/memvalue需修改，否则将造成无限循环。AtomicInteger的incrementAndGet操作即采用了CAS操作。

### Unsafe

为cas的实现提供了底层支持，Unsafe是位于`sun.misc`包下的一个类，主要提供一些用于执行低级别、不安全操作的方法，如直接访问系统内存资源、自主管理内存资源等，这些方法在提升Java运行效率、增强Java语言底层资源操作能力方面起到了很大的作用。但由于Unsafe类使Java语言拥有了类似C语言指针一样操作内存空间的能力，这无疑也增加了程序发生相关指针问题的风险。在程序中过度、不正确使用Unsafe类会使得程序出错的概率变大，使得Java这种安全的语言变得不再“安全”，因此对Unsafe的使用一定要慎重。

unsafe对象为单例对象，只能通过`static getUnsafe()`方法获取，且调用该方法的类需为引导类加载器加载，java -Xbootclasspath/a:${path}将本类添加到引导类加载路径中，或者通过反射调用即可。

unsafe可以操控堆外内存，由于不受GC控制，因此在大内存应用中可以减小GC压力。`DirectByteBuffer`就是一个利用了堆外内存的框架，在Netty、MINA等NIO框架中，

如图示`DirectByteBuffer`的构造函数。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/clipboard (2).png" alt="clipboard (2)" style="zoom:40%;" />

另外，Unsafe还提供了park、unpark方法，用于实现线程的阻塞和唤醒。