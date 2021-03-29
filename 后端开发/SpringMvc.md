1、springmvc是一个集成控制器，模型，视图传递的web开发框架，注重的是业务逻辑的开发，而spring是一个一站式的框架，除了包括mvc架构外，还增加了诸如IOC和AOP的编程模块，支持ORM、Redis等组件，为程序解耦并简洁开发。

2、使用springmvc开发的步骤：

  在web.xml中配置前端控制器（dispatcherServlet），并配置路由映射，即指定相应的路由通过该前端控制器并分发至相应的Controller；并配置contextConfigLocation，指定bean容器即spring.xml的位置。

  在dispatcherServlet配置文件中，声明annotation-config和annotation-driven，使容器可以识别@Controller、@Autowired等注解，并配置controller类的扫描包，配置静态资源处理地址、jsp返回界面地址等。

  接下来就是针对业务的开发了，编写controller控制器层，view、model(service)等处理层。

3、常用注解：

  @Controller，代表控制器，与之常用的@RequestMapping声明访问路径，如果url中含有参数，则使用@RequestParam在路由函数中指明这是一个参数，一般还会声明一个model，用于向前端界面参数对象（通过setattribute的方式）

  @Service，代表服务层，与业务逻辑相关的大量业务都集中在此。

  @Autowired，java bean的DI注解。

  @RequestParam用于将get、post参数绑定到字段中

  @PathVariable用于将url中的字符串当作一个字段传入

  @ModelAttribute用于将数据集合绑定到模型对象中

  @ResponseBody 类名，常用于返回json数据格式的url处理函数，可直接返回类实例，并进行json转换

4、对于请求参数的获取，spring mvc提供了三种方式，一种通过注解的方式获取？后的参数（@RequestParam），一种通过httpservletrequest的形式获取，现代化的方式是通过restful api后的路径直接作文识别为参数（@PathVariable）

5、如何将model返回到前端界面，modelandview 包含视图和模型两个内容，返回view通常以字符串形式查找jsp界面，而model的传递实际上是通过request.setattribute的方式进行了传递。

6、通过binding操作，可以将前端传过来的数据进行模型绑定，通过@ModelAttribute （Course course）将课程信息绑定到一个模型中，进而进行持久化，需注意，表单中name的属性名字需和对象模型的属性名字相同。数据绑定包括基本类型、包装类、数组，前面的通常为一一对应，即名字相同即可，而对于用户自定义类、list、map、set这些需要封装在一个类中，进行类中属性匹配。

7、spring mvc除了可以返回html给人类用户，也可以返回json数据给机器用户，使得前后端分离（restful api开发），通过在视图函数声明@ResponseBody表明该controller会返回json格式的数据。@RequestBody用于接收json数据

**拦截器**

1、拦截器和过滤器运行原理相似，回顾过滤器在web.xml中配置的，对于所有请求都会经过指定的过滤器，进行请求和返回的处理。（编码问题处理等）

2、拦截器的配置实在mvc-dispatcher-servlet中（与web.xml过滤器不同），且需要实现HandlerInterceptor接口，并在配置文件中配置路由规则和相应的处理类

3、interceptor有三个方法，分别是prehandle(有返回值，位false时直接阻断请求)posthandle（请求处理完，返回响应之前执行，可通过modelandview修改传入view的属性对象或者修改返回界面字符串）afterCompletion（响应返回后执行），它们都有Object对象，为请求者的请求目标对象，即controller类。

4、多个拦截器协同工作时，按照注册的顺序执行，pre1->pre2->post2->post1->after2->after1

5、过滤器基于servlet容器，过滤范围较大（包括静态资源），拦截器基于spring容器，仅过滤针对controller的请求。两者都可以减少代码复用，便于维护。