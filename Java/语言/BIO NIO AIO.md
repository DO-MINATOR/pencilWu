### 同步VS异步 阻塞VS非阻塞

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210706102634700.png" alt="image-20210706102634700" style="zoom:50%;" />

同步、异步是针对被调用进程而言，比如文件读写，同步调用，一旦开始IO，则一直等待，直到结果返回。异步调用，开始IO后，结果何时抵达是未知的，只有出现了结果，才取数据。

阻塞、非阻塞是针对调用方而言的，阻塞，是指一旦发起IO，则当前线程就相当于被暂停，cpu依然会分配时间片给该进程，用于检查结果是否返回。而非阻塞，发起IO后，不关心结果何时到达，一般通过回调函数去取结果。

### 通用准则

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210706104655911.png" alt="image-20210706104655911" style="zoom:50%;" />

将任务Runnable或者Callable提交给线程池去执行，返回的Future可以与当前线程进行互动，Runnable不关心结果如何，而Callable任务会有结果返回。主线程每次发起一个IO请求或者被调用一个IO请求后，不会在调用点等待，而是继续其他任务，直到某一时刻再去检查结果是否返回（可以是等一个通知信号，也可以是循环检查），这仍是同步；如果至始都不再主动处理业务了，而是由回调函数handler发起一个新线程自己处理，则是异步。

### BIO模型

基于阻塞调用方式，因此只能为每一个IO开启一个新的线程，以同时服务多个请求。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210707095812460.png" alt="image-20210707095812460" style="zoom:50%;" />

1. 服务端的serversocket.accept()方法是阻塞式的，即等待客户端传来请求才继续。
2. socket.getInputStream和socket.getOutputStream获得的流的IO也是阻塞式的。
3. 因此服务端必须为每一个客户端新开一个线程才能覆盖到服务。
4. 如果一个客户端长期没有响应且不主动结束线程，则服务端也无法主动结束线程，这将导致资源浪费。

### NIO模型

使用Channel代替Stream，并通过缓冲区ByteBuffer进行数据读写。非阻塞体现在单个线程可以创建多个channel并监管它们，具体通过Selector。Channel的双工模式介绍。

- 写模式（channel.read()）

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210707175403433.png" alt="image-20210707175403433" style="zoom:67%;" />

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210707175526050.png" alt="image-20210707175526050" style="zoom:67%;" />

- 读模式(channel.write())

调用flip()切换为读模式

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210707175958779.png" alt="image-20210707175958779" style="zoom:67%;" />

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210707180017586.png" alt="image-20210707180017586" style="zoom:67%;" />

此时调用clear()，恢复为初始写模式。

如果未读取完全，则调用compact()，将剩余数据保存在“顶端”。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210707180518311.png" alt="image-20210707180518311" style="zoom:67%;" />

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210707180618795.png" alt="image-20210707180618795" style="zoom:67%;" />

文件复制对比测试

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210708104211672.png" alt="image-20210708104211672" style="zoom: 50%;" />

文件较小时，BIO效率较高，文件增大时，NIO效率较高。

### channel+selector实现多路复用

注册到selector，通过监管selector来检查channel状态。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210708104426819.png" alt="image-20210708104426819" style="zoom:50%;" />

channel的4种状态：connect、accpet、read、write。

selector的常用操作：

- interestOps()：待监听的状态
- readyOps()：准备好的状态
- channel()：返回指定channel
- selector()：该channel注册的seletor
- attachment()：绑定的任意对象
- keys()：获取所有监听的对象key
- selectedkeys()：获取到有状态发生的对象key

### AIO模型

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210708202046689.png" alt="image-20210708202046689" style="zoom: 67%;" />

之前讲的BIO以及NIO其实本质上都是同步模型，前者是阻塞在调用点，一旦数据准备好后，才返回，后者是第一次调用数据没有准备好，需要适时重新检查数据是否准备好。

注意NIO的N仅和channel有关，因为channel可以通过configureBlocking(false)设置为非阻塞式，即调用后即使数据没有准备好，也能立即返回。而selector起到了一个伪异步的作用，即通过状态改变来通知channel可以进行数据读写。

AIO模型是真正的实现了异步处理的机制，调用方发起请求后，可以不管结果如何而直接返回，数据准备好后也无需关心何时处理。

- Futrue

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210708203745549.png" alt="image-20210708203745549" style="zoom:67%;" />

将IO请求通过Callable执行，并返回一个Future，此为非阻塞，此后调用Future.get()（该方法是阻塞的），因此总体上来看，该模型还是阻塞式同步模型。

- CompletionHandler

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210708204339937.png" alt="image-20210708204339937" style="zoom:67%;" />

这是真正的AIO，发起IO请求的同时，传入一个回调函数，IO完成后调用回调函数进行处理。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210712101743154.png" alt="image-20210712101742870" style="zoom:50%;" />

AIO的底层是通过一个AsyncChannelGroup来管理不同的IO通道，即每次发起一个异步非阻塞的IO请求时，一旦IO事件发生，则启动一个线程来处理该IO，在聊天模型中，服务端发起一个accept请求，之后一旦成功建立连接以后，则内部启动一个线程来执行预先定义好的Handler来处理该IO，返回一个AsyncSocket对应客户端，同样也是非阻塞异步的，read、write方法都可以使用Handler来处理。

### 服务器模型

服务器底层其实也是利用socket进行数据读取和响应，不同的是多了一层应用层协议，常见的比如HTTP服务，就需要解析HTTP请求行、请求头，以及请求体，并将这些封装在ServletRequest中，通过connector逐层传递到具体servlet。

<img src="https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210712171118495.png" alt="image-20210712171118495" style="zoom:50%;" />

- 静态资源
  - 通常和请求行相关，不因请求体不同而不同
  - 请求常见资源如HTML、CSS、GIF等
  - 服务器直接根据资源路径获取文件，并通过socket.getOutputStream获取输出流将数据输出到客户端
- 动态资源
  - 请求到达客户端需要进一步解析（后台逻辑业务的处理）得到结果，再返回
  - 通常以servlet的形式存

在Tomcat中，servlet属于应用层开发人员编写代码，由于也需要用到ServletRequest、ServletResponse等对象，为了避免这部分属于服务器底层的逻辑被外界访问到，在这里使用了一个门面模式(facade)，防止内部逻辑被外界访问。

### 三种模型对比

- BIO：每一个请求都将开启一个新线程，适用于连接数量少，开发、维护较容易，可通过线程池改进
- NIO：所有请求的channel都被一个selector监控，适用于连接数目多，服务器cpu资源较少，即IO多路复用一个线程，每个请求处理不能花费较多时间，否则其他IO准备好后也无法及时处理。
- AIO：IO请求好后，通过AsynchronousChannelGroup开启新的handler来处理，适用于连接数目多，由于是多线程模型，所以相较NIO，请求间不会相互影响。

