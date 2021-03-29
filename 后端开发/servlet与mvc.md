1、servlet先于jsp诞生。基于一种路由配置和请求响应的模式开发的，类似于原生的uwsgi服务器，servlet可以独立于jsp存在，即不用编写webapp目录及其子文件，只需要extends了HttpServlet的类即可，但可以使用webxml文件为servlet初始化配置参数。

2、servlet的装载有三种情况：

  请求自动装载，即用户第一次访问到该servlet的请求时，自动进行servlet的初始化、构造放方法并执行service。

  声明服务启动时装载，使用load-on-startup。

  当servlet修改时，自动装载。

3、servlet存在多线程安全问题，因为访问只会创建一个servlet实例对象，而多个用户访问同一个对象势必存在线程安全问题。在以前老的版本解决方案中，采用的是让serlvet去实现SingleThreadModel接口，这样就会为每一个线程创建一个该servlet实例对象，但本质不是解决多线程问题。

4、servlet内置对象：

  servletconfig：web-xml可以设置servlet初始化时的参数，而该对象可以获取这些初始化的参数，在servlet中使用getInitparameter获取初始化参数。

  ServletContext：多个serlvet共享一个上下文，可以实现数据共享。也可实现参数获取、请求转发操作；读取资源文件，设置有效缓存时间。

5、服务端使用outputstream或者printwriter向客户端输出信息，都需要注意编码问题，前者输出byte类型，后者可直接输出字符串。另外，下载文件应使用outputstream字节流，字符流会丢失二进制数据。

6、在Model 1基础上多了servlet的模型称之为Model 2，由此引入了MVC的设计模式，如图，servlet充当控制器，返回页面由jsp提供，模型层（业务逻辑以及数据对象）封装在javabean中。

![image-20200530101836583](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200530101836583.png)

7、mvc的设计流程是，根据业务逻辑先设计出合理的模型层，用于表示业务对象所拥有的属性和方法，然后测试改模型所有方法以及属性返回值。然后在servlet中根据客户端的请求实施相应业务逻辑，在最终返回给用户前端jsp页面时，注意传入正确模型，jsp再通过脚本代码获取模型数据。