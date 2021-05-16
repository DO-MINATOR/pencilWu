### 三大类

- ServletContext（服务器对象，生命周期和Application一致）
  - ServletContextListener：监听ServletContext生命周期（创建到销毁）,web中加载spring的监听器ContextLoaderListener就实现该类
  - ServletContextAttributeListener：监听ServletContext域中属性变化的。

- HttpSession（客户与服务器的长久连接对象）
  - HttpSessionListener：httpsession的生命周期（第一次获取创建，自动超时/手动销毁，服务器关闭只会自动钝化）
  - HttpSessionAttributeListener：httpsession域中属性变化
  - HttpSessionActivationListener：监听某个对象在session中的活化/钝化
  - HttpSessionBindingListener：监听某个对象在session中的保存和移除
- ServletRequest
  - ServletRequestListener：request的生命周期监听
  - ServletRequestAttributeListener：request域中属性变化

### 配置

属于tomcat组件，因此同servlet和Filter一样在web.xml中配置。

ServletContext和HttpSession需要在web.xml中进行配置

```xml
<listener>
    <listener-class>...</listener-class>
</listener>
```

同时创建的类需要实现ServletContextListener、ServletContextAttributeListener以及HttpSessionListener、HttpSessionAttributeListener。**注意：**HttpSessionActivationListener、HttpSessionBindingListener监听的是某个对象的活化、钝化以及保存、移除，因此只需要在bean对象中上实现该类即可。

### 使用场景

- ServletContextListener：服务器启动时的初始化工作，以及停止后的清理工作。
- HttpSessionAttributeListener/HttpSessionBindingListener：可用于监听用户登陆后将Userbean添加到session属性域