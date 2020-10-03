1、AQS是各种锁以及线程协作类的实现基础。在ReentryLock、Semaphore、ReentryReadWriteLock中都包含了一个sync内部类，该类继承AQS（AbstractQueuedSynchronizer）

![image-20200527162242995](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527162237792.png)

2、AbstractQueuedSynchronizer定义了三个属性：

  State：volatile修饰，原子操作，在不同的协作类中意义不同，Semaphore（剩余资源量）ReentryLock（重入次数）、CountdownLatch（倒数计数）

  获取、释放state方法：依据不同协作类有具体的实现

  竞争、等待队列：FIFO队列，用于存放等待线程，当线程竞争失败时，会进入等待队列，释放后，挑选合适的线程来执行。

3、以CountdownLatch为例，

在CountdownLatch的构造函数中，会初始化Sync，并为Sync中的State赋值（该state此时为需要倒数的计数）

![image-20200527162237792](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527162242995.png)

在CountdownLatch中的await方法中，会调用sync的acquireSharedInterruptibly ()方法

![image-20200527162250023](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527162250023.png)

最终会调用到CountdownLatch特有的tryAcquireShared()方法，如果判定state不等于0，则会调用doAcquireSharedInterruptibly()将该线程加入阻塞队列。release方法同理。

![image-20200527162254910](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527162254910.png)
