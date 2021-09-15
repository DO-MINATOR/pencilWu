### 线程与协程

线程是CPU调度的最小单位，分为KLT和ULT模型，JVM使用KLT模型，线程由CPU直接管理，可以最大限度的利用CPU核，线程和OS线程保持1：1关系。

协程更为轻量级，由用户空间管理，多个协程运行在一个线程空间下，多个协程在一个核下运行的效率高于多个线程运行在一个核下，因为不同协程的切换并不涉及用户态、核心态的切换，也没有线程栈空间的保存与恢复，只是同一个线程下不同协程逻辑的调用。

`注：`java原生不支持协程，因为是KLT模型。

### 线程池

线程是稀缺资源，创建、阻塞、唤醒操作十分昂贵，因此不建议频繁的创建线程，而是保留一部分线程，等待作业的到来，去执行新任务。java提供了一套创建、分配、监控线程池的方法。

应用场景，web业务中，如果并发请求量非常大，但执行时间较短，就会频繁的创建线程，使得非业务代码的执行时间很长。适用于任务数量多，且单个任务执行时间短的场景。

![diagram](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/diagram.png)

- execute()：执行runnable任务。
- submit()：执行runnable任务，也可以提交task，并返回Future对象。
- shutdown()：完成已提交的任务，不再接受新的任务。
- shutdown()：不接受新任务，去除队列的任务，中断正在执行的任务。

### 五种状态

1. RUNNING，正常运行
2. SHUTDOWN，不接受新任务，但能处理现有任务。
3. STOP，不接受新任务，中断正在执行任务。
4. TIDYING，所有任务执行完后，且调用了shutdown方法，则会执行terminated方法。
5. TERMINATED，执行完terminated方法后变为该状态。

![clipboard](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/clipboard.png)

### 创建

**corePoolSize**

线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize；如果当前线程数为corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果执行了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有核心线程。

**maximumPoolSize**

线程池中允许的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize；

**keepAliveTime**

线程池维护线程所允许的空闲时间。当线程池中的线程数量大于corePoolSize的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了keepAliveTime；

**unit**

keepAliveTime的单位；

**workQueue**

用来保存等待被执行的任务的阻塞队列，且任务必须实现Runable接口，在JDK中提供了如下阻塞队列：

- 1、ArrayBlockingQueue：基于数组结构的有界阻塞队列，按FIFO排序任务；
- 2、LinkedBlockingQuene：基于链表结构的阻塞队列，按FIFO排序任务，吞吐量通常要高于ArrayBlockingQuene；
- 3、SynchronousQuene：一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态；
- 4、priorityBlockingQuene：具有优先级的无界阻塞队列；

**threadFactory**

它是ThreadFactory类型的变量，用来创建新线程。默认使用Executors.defaultThreadFactory() 来创建线程。

**handler**

线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了4种策略：

1. AbortPolicy：直接抛出异常，默认策略；
2. CallerRunsPolicy：用调用者所在的线程来执行任务；
3. DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务；
4. DiscardPolicy：直接丢弃任务；

当然也可以根据应用场景自定义策略，如记录日志或持久化不能处理的任务。

**Executors创建方法**

- newFixedThreadPool：core、max固定，使用linkedblockingqueue，可能导致任务数量巨大无法及时处理，造成OOM

- newSingleThreadExecutor：和前者类似，只是core、max都固定为1

- newCachedThreadPool：core为0，max无限大，使用SynchronousQueue，导致一旦有任务，就立即创建线程，也容易造成OOM

- newScheduledThreadPool：可以周期的传入任务，使用DelayedWorkQueue

![image-20200914101230471](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200914101230471.png)

### 监控

```java
public long getTaskCount() //线程池已执行与未执行的任务总数
public long getCompletedTaskCount() //已完成的任务数
public int getPoolSize() //线程池当前的线程数
public int getActiveCount() //线程池中正在执行任务的线程数量
```

### 执行过程

![image-20200527161758476](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527161758476.png)

1. 如果workerCount < corePoolSize，则创建并启动一个线程来执行新提交的任务；
2. 如果workerCount >= corePoolSize，且线程池内的阻塞队列未满，则将任务添加到该阻塞队列中；
3. 如果workerCount >= corePoolSize && workerCount < maximumPoolSize，且线程池内的阻塞队列已满，则创建并启动一个线程来执行新提交的任务；
4. 如果workerCount >= maximumPoolSize，并且线程池内的阻塞队列已满, 则根据拒绝策略来处理该任务, 默认的处理方式是直接抛异常。

### 超时策略

在非核心线程执行完一个任务后，调用getTask()获取下一个任务时，会采取poll(int Delay)带延迟参数的方法，如果任务队列一直没有任务进来，则会返回null，借此可以销毁该线程。

### 异常处理

如果线程池中的任务抛出了异常`throw new RuntimeException`，线程池会创建出新的线程来执行任务，即线程池内部线程资源保持。注意后面的延迟线程池，线程抛异常，虽然也会创建新线程，但任务不会再往队列放。

### ThreadLocal

- 每个线程需要使用同一个对象，为避免多线程造成共享数据出错，方案一：每个线程创建不同对象，开销大；方案二：使用同一个对象，且synchronized保护起来，效率低；方案三：通过ThreadLocal获取对象，本质上为不同线程创建不同对象，但对象数量与线程数量保持一致，即只隔离不同线程使用的对象，不隔离同一个线程下不同任务使用的对象。该场景下threadlocal需要重写initialValue()以确保get时能拿到对象。

- 参数多次传递麻烦，threadlocal可以直接获取该线程下所拥有的信息。该场景下使用threadlocal.set在需要传递对象的情况设置值，通过get获取对象

- 每个线程拥有一个ThreadLocalMap，存放形如<ThreadLocal,Value>的键值对（因为每个线程可以拥有多个ThreadLocal，所以是map，而threadlocal本身作为key）


![image-20200527161805579](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527161835089.png)

- ThreadLocal容易导致内存泄露问题，因为线程池中的线程基本上长时间保持在线，那么其拥有的ThreadLocalMap就会保持，此时如果前面的任务创建了Threadlocal，并且没有释放，那么后面的任务既无法清除掉threadlocal对应的value，也无法使用这片内存，这就导致了内存泄漏。因此，好的做法是，任务在要结束前，主动remove掉自己在线程map中申请的threadlocal，key、value都==null后，GC就会释放掉空间。
