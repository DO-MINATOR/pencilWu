1、过滤器是一个服务端组件，它可以截取用户的请求和相应信息，并根据一定规则进行信息过滤。

2、过滤器设计步骤：创建实现了Filter接口的自定义过滤器，实现init、destroy和doFilter方法，在web.xml文件中配置过滤器名称和class类名，配置url映射（及过滤器响应哪些url的请求）

3、多个过滤器（如果是同一个url映射），则可以组成过滤器链，通过doChain方法依次调用不同过滤器的doFilter方法，类似一种函数调用的堆栈方式。

4、Filter分为6大类，request和include类似（一般请求），forward、error和async。

5、servlet2.5升级到3.0版本后，可无需在web.xml进行配置，而是使用注解声明一个类class是过滤器。

6、pattern映射规则，/*包括所有路由，但不包含.do .jsp结尾的路由，*.do *.jsp匹配这些结尾的路由，/包括所有

