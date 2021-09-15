### AQS简介

java的juc包中的大多数同步器都有着相似的行为，如等待队列、条件队列、独占获取、共享获取，将这些行为抽象出来封装成了一个`AbstractQueuedSynchronizer`对象，即是定义了一套访问共享资源的同步框架。

### 加锁的一般过程-ReentrantLock

```java
ReentrantLock lock = new ReentrantLock(false);//false为非公平锁，true为公平锁
3个线程
T0 T1 T2
lock.lock() //加锁
    while(true){
        if（cas加锁成功){//cas->比较与交换compare and swap，
            break;跳出循环
        }
        //Thread.yeild()//让出CPU使用权
        //Thread.sleep（1）;
        HashSet，LikedQueued(),
        HashSet.add(Thread)
            LikedQueued.put(Thread)
        阻塞。
        LockSupport.park();
	}
    
    T0获取锁
    xxxx业务逻辑
    xxxxx业务逻辑
    
lock.unlock() //解锁
Thread  t= HashSet.get()
Thread  t = LikedQueued.take();
LockSupport.unpark(t)；
```

加锁过程需要保证的点

- 自旋/阻塞：当其余线程加锁失败后，须停在加锁逻辑环节，通常是调用lockSupport.park()方法使当前线程阻塞。
- CAS：加锁逻辑保证只有一个线程能成功获取到锁，cas设值state变量
- queue队列：未加锁成功的线程进入同步等待队列，等待被唤醒。

### AQS调用逻辑

#### 三大属性

- state：在不同的线程协作类中有不同意义，Semaphore（剩余资源量）ReentryLock（重入次数）、CountdownLatch（倒数计数）。
- exclusiveOwnerThread：当前获取到锁的线程
- queue：双向队列，未加锁成功的线程被放到该队列中，等待被唤醒。

#### 调用过程

在CountdownLatch的构造函数中，会初始化Sync，并为Sync中的State赋值（该state此时为需要倒数的计数）

![image-20200527162237792](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527162242995.png)

在CountdownLatch中的await方法中，会调用sync的acquireSharedInterruptibly ()方法

![image-20200527162250023](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527162250023.png)

最终会调用到CountdownLatch特有的tryAcquireShared()方法，如果判定state不等于0，则会调用doAcquireSharedInterruptibly()将该线程加入阻塞队列。release方法同理。

![image-20200527162254910](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527162254910.png)

自定义同步器只需重写state的获取和释放，即`tryAcquire()`和`tryRelease()`方法，不同同步器有不同的获取和释放逻辑，因此这两个方法的业务就是如何变动state，如果返回true，则代表加锁成功，返回false，加锁失败，AQS底层会自动将线程阻塞并放入到同步等待队列中；同样的，如果解锁成功，则唤醒同步等待队列中的第一个结点。

### 同步等待队列CLH

一种FIFO先入先出线程等待队列，为阻塞机制。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/clipboard.png" alt="clipboard" style="zoom:67%;" />

### 条件等待队列

ReentrantLock中的Condition，调用await方法后，会将当前线程放入到条件等待队列中(该队列中的线程全部都是等待signal信号唤醒的线程)，同时唤醒CLH队列中的第一个结点。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/clipboard (1).png" alt="clipboard (1)" style="zoom:67%;" />

### BlockingQueue

一种Concurrent包下提供的用于解决生产者、消费者问题的队列，特性是只有一个线程能对该队列进行出队和入队操作，且当队列满或空时，无法进行入队和出队操作。除了解决生产消费问题，在线程池中也有应用。基于ReentrantLock保证线程安全，基于Condition协作生产、消费，同时用到同步等待队列和条件等待队列。

- CLH：锁机制保证线程安全
- 条件等待队列：当条件不满足时，存放到该队列中，防止线程被唤醒，条件满足时，才会移到CLH中，等待被唤醒。

