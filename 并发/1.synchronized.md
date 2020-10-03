1、synchronized是java原生支持的最基本的互斥手段，即保证一段代码的原子性（同一时刻只有一个线程执行）

2、Synchronized分为：

  对象锁：方法锁（锁对象为this），代码块锁（指定不同锁对象）

方法锁

```java
public synchronized void method() {
    ...
}
```

代码块锁可以自定义多个锁对象，适用于业务逻辑复杂的场合。

```java
synchronized (this) {
    ...
}
```

  类锁：静态方法锁（锁对象为class本身），代码块锁（*.class）

静态方法锁

```java
static synchronized void method(){
    ...
}
```

代码块锁

```java
synchronized (StaticFunctionblock.class){
    ...
}
```

**线程在运行过程中，抛出了异常，即使未被捕获，（此时JVM进行捕获并强制中断该线程），那么该线程所占用的锁也会被释放。**

3、性质：

可重入锁，synchronized代码中外层获得该锁后，内部任意代码都可以直接使用该锁。

不可中断，遇到synchronized修饰的代码块，如果无法获取到该锁，那么进入到blocked状态，且该状态无法响应interrupt。