1、原子操作，同一时刻，一组操作只能由一个线程执行，且执行过程中，内部逻辑不能被其他线程打破。

2、原子类，java提供的能够保证自身是原子操作的类，相比锁机制：

  a、粒度更小，提升系统效率

  b、如果发生特殊情况（高度竞争）则会由于乐观锁或者自旋导致cpu占用率飙升

3、常用的原子类

![image-20200527161950228](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527161950228.png)

基本类型：

  AtomicInteger：get\getAndSet\getAndIncrement\getAndDecrement\getAndAdd\comapreAndSet，以上只有get操作无锁，其余方法都是通过自旋操作实现的。

**注：compareAndSwap是自旋的本质，即线程判断当前值并设置新的值，这样一旦一个线程自旋判断成功，则其余线程将自旋直至条件满足。**

  AtomicIntegerArray：数组中每个元素都是AtomicInteger

  AtomicReference：可以传入泛型，参考自旋锁的实现。

  AtomicIntegerFieldUpdater：可以将某个不支持原子操作的变量升级为支持原子操作。

  AtomicLong Vs LongAdder：当多个线程对AtomicLong进行+1操作时，由于竞争同一资源，因此大部分线程都会陷入自旋中，且每次执行完+1操作后，都要进行一道flush和refresh操作（以供其他核心读取到正确的值）这样当竞争激烈到一定程度后，就会浪费大量的CPU资源（活锁），这时候可采取的方法有二，其一是退化到原来的悲观锁，即每次执行加解锁，这样可以腾出cpu时间给其余线程，其二是继续采用原子类，采用更加高级的LongAdder，但检测到激烈程度过大后，就会将不同线程分小组，每个线程各自维护本组的自增变量，这样多个核心的线程可以同时工作，最后将得到的结果进行汇总。

  LongAccumulator：LongAdder可以实现多个核心同时执行+1操作，而LongAccumulator可以实现更复杂的表达式计算(不能依赖顺序)，这样，即使是对同一共享资源的访问，也可以实现多线程并发执行。

```java
LongAccumulator longAccumulator = new LongAccumulator((x, y) -> x + y, 0);
ExecutorService executorService = Executors.newFixedThreadPool(20);
IntStream.range(1, 1000).forEach(i -> {
    executorService.submit(() -> {
        longAccumulator.accumulate(i);
    });
});
```

4、CAS（CompareAndSwap）是乐观锁、并发容器、原子类的实现核心

CAS是CPU原语，默认保证了原子操作，即多个线程只有一个能够执行CAS操作，CAS包含三个参数（memValue，expectValue，newValue），第一个为变量在内存中的值（由于没有锁的可见性保证，所以多个线程要想及时访问到变量真实的值，就只能通过unsafe提供的内存变量直接读取操作），之后其中一个线程成功修改了该变量，其余变量之后执行都会返回false，可以根据返回的布尔值决定退出线程还是while重试，如果采取重试策略，要注意expectValue/memvalue需修改，否则将造成无限循环。AtomicInteger的incrementAndGet操作即采用了CAS操作。

CAS的弊端是如果采用重试策略，那么竞争程度不能过于激烈，且同步代码区不能过冗长，否则还是建议采取锁机制，好让出cpu资源。