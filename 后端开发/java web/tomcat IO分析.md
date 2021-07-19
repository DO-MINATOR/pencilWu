### Connector

```java
public class Connector implements Runnable {

    private static final int DEFAULT_PORT = 8888;

    private ServerSocketChannel server;
    private Selector selector;//基于channel和selector的NIO模型
    private int port;

    public Connector() {
        this(DEFAULT_PORT);
    }

    public Connector(int port) {
        this.port = port;
    }

    public void start() {
        Thread thread = new Thread(this);
        thread.start();
    }

    @Override
    public void run() {
        try {
            server = ServerSocketChannel.open();
            server.configureBlocking(false);
            server.socket().bind(new InetSocketAddress(port));

            selector = Selector.open();
            server.register(selector, SelectionKey.OP_ACCEPT);//ServerSocketChannel绑定accept事件到selector上
            System.out.println("启动服务器， 监听端口：" + port + "...");

            while (true) {//轮询所有发生状态改变的key及其对应通道
                selector.select();
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                for (SelectionKey key : selectionKeys) {
                    // 处理被触发的事件
                    handles(key);
                }
                selectionKeys.clear();
            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            close(selector);
        }
    }

    private void handles(SelectionKey key) throws IOException {
        // ACCEPT
        if (key.isAcceptable()) {
            ServerSocketChannel server = (ServerSocketChannel) key.channel();
            SocketChannel client = server.accept();
            client.configureBlocking(false);
            client.register(selector, SelectionKey.OP_READ);//SocketChannel绑定read事件到selector上
        }
        // READ
        else {
            SocketChannel client = (SocketChannel) key.channel();
            key.cancel();//socket的inputstream和outputstream不支持异步处理，因此改为block模式，同时取消监听该通道，相当于一次socket连接只处理一次http请求
            client.configureBlocking(true);
            Socket clientSocket = client.socket();
            InputStream input = clientSocket.getInputStream();
            OutputStream output = clientSocket.getOutputStream();

            Request request = new Request(input);//解析请求行，请求头，封装在request对象中
            request.parse();

            Response response = new Response(output);
            response.setRequest(request);

            if (request.getRequestURI().startsWith("/servlet/")) {//处理动态资源请求
                ServletProcessor processor = new ServletProcessor();
                processor.process(request, response);
            } else {
                StaticProcessor processor = new StaticProcessor();//处理静态资源请求
                processor.process(request, response);
            }

            close(client);
        }
    }

    private void close(Closeable closable) {
        if (closable != null) {
            try {
                closable.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

connector连接器负责监听来自客户端的http请求，基于channel和selector的NIO模型，每次新的连接进来后，绑定read事件到selector上，socket的inputstream和outputstream是阻塞式的，因此当read事件触发后，需要手动调用`key.cancel()`解除selector的监听，同时设置为block模式`client.configureBlocking(true)`，read事件触发后首先做的就是解析协议字段，包括http请求行，请求头，并封装在请求对象request中，之后再传到tomcat容器进行逐层处理，根据请求路径最终找到静态资源或动态资源进行处理。

### Request

```java
public class Request implements ServletRequest {

    private static final int BUFFER_SIZE = 1024;

    private InputStream input;
    private String uri;

    public Request(InputStream input) {
        this.input = input;
    }

    public String getRequestURI() {
        return uri;
    }

    public void parse() {
        int length = 0;
        byte[] buffer = new byte[BUFFER_SIZE];
        try {
            length = input.read(buffer);
        } catch (IOException e) {
            e.printStackTrace();
        }

        StringBuilder request = new StringBuilder();
        for (int j = 0; j < length; j++) {
            request.append((char) buffer[j]);
        }
        uri = parseUri(request.toString());
    }

    private String parseUri(String s) {
        int index1, index2;
        index1 = s.indexOf(' ');
        if (index1 != -1) {
            index2 = s.indexOf(' ', index1 + 1);
            if (index2 > index1) {
                return s.substring(index1 + 1, index2);
            }
        }
        return "";
    }

    @Override
    public Object getAttribute(String s) {
        return null;
    }

    @Override
    public Enumeration getAttributeNames() {
        return null;
    }

    @Override
    public String getCharacterEncoding() {
        return null;
    }

    @Override
    public void setCharacterEncoding(String s) throws UnsupportedEncodingException {

    }

    @Override
    public int getContentLength() {
        return 0;
    }

    @Override
    public String getContentType() {
        return null;
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        return null;
    }
	//......
}
```

Request对象包含了常见属性如InputStream，保存来自socket的getInputStream，常见协议属性uri、contentLength，通过调用inputstream.read()方法并进一步解析，将这些请求行、请求头信息保存在这些字段中。

### Response

```java
public class Response implements ServletResponse {

    private static final int BUFFER_SIZE = 1024;

    Request request;
    OutputStream output;

    public Response(OutputStream output) {
        this.output = output;
    }

    public void setRequest(Request request) {
        this.request = request;
    }

    public void sendStaticResource() throws IOException {
        File file = new File(ConnectorUtils.WEB_ROOT, request.getRequestURI());
        try {
            write(file, HttpStatus.SC_OK);
        } catch (IOException e) {
            write(new File(ConnectorUtils.WEB_ROOT, "404.html"), HttpStatus.SC_NOT_FOUND);
        }
    }

    private void write(File resource, HttpStatus status) throws IOException {
        try (FileInputStream fis = new FileInputStream(resource)) {
            output.write(ConnectorUtils.renderStatus(status).getBytes());
            byte[] buffer = new byte[BUFFER_SIZE];
            int length = 0;
            while ((length = fis.read(buffer, 0, BUFFER_SIZE)) != -1) {
                output.write(buffer, 0, length);
            }
        }
    }

    @Override
    public String getCharacterEncoding() {
        return null;
    }

    @Override
    public ServletOutputStream getOutputStream() throws IOException {
        return null;
    }

    @Override
    public PrintWriter getWriter() throws IOException {
        PrintWriter writer = new PrintWriter(output, true);
        return writer;
    }

    @Override
    public void setContentLength(int i) {

    }

    @Override
    public void setContentType(String s) {

    }

    @Override
    public void setBufferSize(int i) {

    }

    @Override
    public int getBufferSize() {
        return 0;
    }

    @Override
    public void flushBuffer() throws IOException {

    }
	//......
}
```

Response对象主要负责想客户端发送数据到outputStream中，对于静态资源，根据request的uri字段，获取静态资源流，再发送给客户端，而动态资源，则是直接对OutputStream之上封装一层PrintWriter，获取输出流，更方便servlet进行数据输出，由于有缓冲区，因此需要手动flush。

### StaticProcessor

```java
public class StaticProcessor {

  public void process(Request request, Response response) {
    try {
      response.sendStaticResource();
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```

静态资源处理器，可以看作tomcat容器的精简版，直接根据request的uri获取原始资源流，并通过response返回数据。

### ServletProcessor

```java
public class ServletProcessor {

    URLClassLoader getServletLoader() throws MalformedURLException {
        File webroot = new File(ConnectorUtils.WEB_ROOT);
        URL webrootUrl = webroot.toURI().toURL();
        return new URLClassLoader(new URL[]{webrootUrl});
    }

    Servlet getServlet(URLClassLoader loader, Request request) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
    /*
     /servlet/TimeServlet
    */
        String uri = request.getRequestURI();
        String servletName = uri.substring(uri.lastIndexOf("/") + 1);

        Class servletClass = loader.loadClass(servletName);//调用URLClassLoader.loadclass方法加载对应servlet
        Servlet servlet = (Servlet) servletClass.newInstance();
        return servlet;
    }

    public void process(Request request, Response response) throws IOException {
        URLClassLoader loader = getServletLoader();
        try {
            Servlet servlet = getServlet(loader, request);//根据request对象获取对应servlet
            RequestFacade requestFacade = new RequestFacade(request);
            ResponseFacade responseFacade = new ResponseFacade(response);
            servlet.service(requestFacade, responseFacade);//调用最终方法servlet.service执行业务并返回数据
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (ServletException e) {
            e.printStackTrace();
        }
    }
}
```

动态资源处理器，也可以看作tomcat容器的精简版，流程如注释所示。这里的URLClassLoader可以进一步改进，并不需要每次都去重新加载一遍该servlet对应的类，否则会造成元空间溢出，可以先判断该加载器是否已经加载过该类，如果已经加载，则直接返回对应实例，后台启动一个守护线程，定期检查servlet.class是否发生变化，以实现服务器热加载。

### TimeServlet

```java
public class TimeServlet implements Servlet {

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        PrintWriter out = servletResponse.getWriter();
        out.println(ConnectorUtils.renderStatus(HttpStatus.SC_OK));
        out.println("What time is it now?");
        out.println(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
                .format(new Date()));
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {

    }
}
```

这里以TimeServlet作为示例说明动态资源如何返回数据，着重强调facade设计模式，由于request有些方法属于服务器组件自身调用的方法，如协议解析`parse()`，这些方法并不想被servlet这些应用层方法，因此提供一个门面对象，对于request，提供RequestFacade，同样实现了ServletRequest接口，对于一些常见方法，只进行简单包装，因为没有了parse()等方法，因此传入给servlet的request对象即使向下转型也无法调用到这些方法。

### NIO

注意在Connector中，通过selector监听多个通道的事件，注意read事件，由于触发第一次read后，就立即解绑了该事件，因此每一个socket连接只能处理一次http请求，这样就无法利用http1.1的长连接特性，每次响应头中的属性keep-alive设值为close，这样客户端就需要新开启一个connect连接，继而发送新的资源请求。

在该模型中，多路复用导致了多个请求到达服务器时无法并发执行，只能通过selector轮询调度，大大降低了服务器的响应能力，为了改进，可以采用原始的BIO+线程池的模型，每个线程可以设置readTimeOut，最大长连接数量，即一个socket所能允许的最大的http请求数量。也可以改用AIO模型，当请求accpet事件触发时，调用handler执行相关逻辑，主要是准备第一次获取数据，执行read方法和相关handler，该handler中，也可以根据是否keep-alive来选择是否继续在该连接中执行read方。

目前为止，只有BIO+线程池和AIO的模型支持http并发请求，而NIO由于是多路复用，因此无法支持并发访问。

**BIO访问模式：**

![image-20210713193959609](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210713193959609.png)

**NIO访问模式：**

![image-20210713195529112](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210713195529112.png)

NIO也可以实现并发请求，之前说无法并发访问是因为selector在轮询事件发生后，是依次对准备好的事件进行处理，如果将该调用逻辑开启一个新线程处理的话，即可将就绪的事件并发处理，另外，还可以开启多个poller线程，每个poller内部有一个selector，将事件按照规则绑定到其中一个selector上，这样也可以并发轮询。

### 长连接

长连接是http1.1支持的新特性，允许多个请求复用一个socket连接，减小了服务器每次创建、维护socket的压力，长连接适用于以上三种模型，socket的前一次http请求处理完毕后，可以根据请求头的keep-alive属性或者服务器自身配置设置响应头中的keep-alive属性，最终长连接是否生效，取决于响应头中的keep-alive属性，如果为true，则浏览器再发送下一次http请求时，会复用之前已经建立好连接的socket，而对于服务端，再处理下一次请求时，需要对recvbuf中尚未处理的数据做处理（比如请求体中的剩余数据），通常时是直接覆盖，将下一次的http请求行、请求头和请求体直接从最近一次处理的buf下标进行覆盖。

### 异步servlet

servlet3.0开始支持异步处理，原本的处理方式如下图所示，当acceptor收到一个http请求后，将该请求发送给一个socketProcessor进行处理，如果这个处理逻辑比较花时间，譬如IO密集型，则这个socketProcessor将会长期占用线程池资源，减少了并发量。

![Servlet 的主要工作流程](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/20180923210340371)

改良的做法是，如果发现需要做一些费时间的操作，则可以在servlet中创建一个Runnable()，这其中负责执行相应逻辑，并调用`servletRequest.startAsync()`开启一个异步类，调用start()方法执行该逻辑。这样socketProcessor可以立即返回容器，等待其他请求。

![Servlet3.0 处理流程](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/20180923212032240)

```java
AsyncContext asyncContext = servletRequest.startAsync(request,response);
asyncContext.start(new Runnable() {
    @Override
    public void run() {
        //...
        asyncContext.complete();
    }
});
```

需要说明的是，异步挂起并不会释放掉该socket，且因为客户端和服务器的请求逻辑也不会在该socket处理完之前向同一个socket再次发送请求。只有调用`asyncContext.complete()`方法后才会提示该socket处理完毕，整个异步线程也就处理完毕。

