1、jsp是java server page（简化版的servlet）实现了在java语言中编写html标签，它俩都是在服务端执行的，jsp的运行需要容器服务器的支持，例如tomcat或者apache，它们负责截获rul请求，并将响应信息返回给前端，同时由于内嵌了jsp容器，因此可以识别后端的jsp脚本语言并执行，即java代码

2、<% ... %>在内部使用jsp脚本以执行。

3、声明语句为<%! ... %>，表达式<%= ... %>（内部引用声明的变量或方法）

4、jsp的生命周期：对于第一次请求，生成jsp对应的servlet（字节码文件），并执行jspinit初始化；然后字节码文件中执行jspService，用于处理用户的请求并响应。

5、jsp包含三种指令，page，定义脚本编码，使用的脚本语言（java）等；include，包含的其他文件；taglib，页面自定义属性。

6、行为标签，用于控制servlet引擎

![image-20200530101624226](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200530101624226.png)

jsp：include是运行时包含，而前面的include是装载成servlet时进行的包含

7、jsp内置对象，不需要使用new就可以直接使用。

out常用方法为println，直接向客户端输出字符串

request用于获取请求参数，get、post均可，常用getParameter方法：以及getContentType()、getProtocol()、getServerName()等常用方法。

reponse的printWriter对象总是比out对象先输出至客户端，除非out使用flush强制输出客户端；sendRedirect（请求重定向是客户端行为，302显示改变URL地址，而请求转发requestDispatcher是服务端行为，url不会改变，转发可以保存请求信息）

session是服务端用于保存客户端用户信息的机制，当客户端第一次打开一个jsp开始，服务端创建一个对应的session，直至用户完全关闭所有有关该服务器的会话，或者session到期。

application是用于保存全局应用服务器状态信息的机制，所有客户端都可以获取到同一个application信息。

pageContext对象作用域为当前页面，退出即清空。

pageContext是上述所有对象的一个集大成者，可以获取到其他任何对象。

exception对象，当一个页面抛出异常时，需要有一个单独的页面接受处理该异常，发生异常的页面需要指定当异常发生时跳转到的界面，而接受异常的界面需要设置isErrorpage属性为true，然后使用该内置对象获取异常信息。

8、Javabean，设计原则是公有类、无参公有构造方法、属性私有，getter、setter方法。使用方法：  用java中的new，通过getset方法操作，需要import导入。使用jsp动作标签，jsp:usebean，id指定实例名称，class指代引用的包，scpoe默认page。

![image-20200530101701423](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200530101736242.png)

同理，使用getproperty可以获取javabean实例的属性值，方法和上述类似。

9、关于javabean的作用域，从小到大分别为page、request、session、application，可以使用getproperty或者getattribute获取。

10、javabean的出现引入了Model 1模式，在此之前全部的业务逻辑、数据封装和视图呈现全部由jsp完成，而javabean则取代了部分的业务逻辑和数据封装。

![image-20200530101736242](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200530101748853.png)

11、cookie和session都可以用于保存和读取用户信息，不同之处在于cookie是保存在客户端，而session保存在服务端，一般关闭浏览器就会自动清除。

12、指令与动作

page指令：脚本语言、导入的Java包、页面字符集

include指令与include动作区别如图：

![image-20200530101748853](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200530101754184.png)

taglib指令：用于自定义页面属性。

forward动作：和request的请求转发相同。

param动作：常和foward动作一起使用

![image-20200530101754184](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200530101701423.png)