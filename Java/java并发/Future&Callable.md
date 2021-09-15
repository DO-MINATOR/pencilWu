1、Runnable有两大缺陷，分别是无返回值，无法抛出异常。

2、使用Callable接口可以解决该问题，而Future可以获取callable信息，通过future.get获取返回值，future.isDone判断任务是否执行结束。

3、步骤

  创建callable对象

```java
Callable<Integer> callable = () -> {
    Thread.sleep(2000);
    return 21;
};
```

  通过线程池.submit提交，Future<T> future作为返回值

  调用future.get()阻塞等待任务返回值。

4、future任务无论内部抛出什么异常，父线程接受到的异常永远是ExecutionException，并且只有显示调用get方法才会接收到异常。isDone仅代表任务完成，抛出异常也算。当get方法显示传入时间时，如果超时未获取到结果则catch(TimeoutException e)。调用cancel(true)方法可以取消子任务继续执行，true代表发送interrupt信号，此时再调用get将会抛出cancellationException

5、cancel传入false对正在执行的线程不起作用，仅针对还未启动的任务，这样线程就不会执行。

![image-20200527162213446](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200527162213446.png)