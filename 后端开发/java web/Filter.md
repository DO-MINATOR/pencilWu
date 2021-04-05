5、servlet2.5升级到3.0版本后，可无需在web.xml进行配置，而是使用注解声明一个类class是过滤器。

6、pattern映射规则，/\*包括所有路由，但不包含.do .jsp结尾的路由，*.do *.jsp匹配这些结尾的路由，/包括所有

### 简介

1. 三大组件之一：Servlet、Listener、Filter
2. 作为JavaEE过滤器的接口
3. 作用是拦截请求，过滤响应(如：权限检查、日记操作、事务管理)

图示说明请求和响应的过滤

![image-20210405111653407](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210405111653407.png)

### 实现步骤

1. 编写一个实现Filter的类。
2. 实现DoFilter方法，Filterchain.doFilter()方法可以让请求正常继续。
3. 在web.xml中配置该Filter并配置过滤地址。

### 生命周期

1. 构造器
2. init初始化
3. doFilter执行过滤动作，请求正常响应或下一个Filter
4. destroy停止web工程执行

### 过滤器链

![image-20210405113356836](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210405113356836.png)

### 匹配方式

-  精确匹配

  ```xml
  <url-pattern>/admin/target.jsp</url-pattern>
  ```

- 目录匹配

  ```xml
  <url-pattern>/admin/</url-pattern>
  ```

- 后缀匹配

  ```xml
  <url-pattern>*.jsp</url-pattern>
  ```

-  全匹配

   ```xml
   <url-pattern>/*</url-pattern>
   ```

**注意：**过滤器不论请求的资源是否存在，只管路径是否匹配。

