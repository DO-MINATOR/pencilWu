### EL

EL表达式：Expression Language，表达式语言，用于替代JSP表达式进行数据输出，当然后台逻辑还是由servlet执行。

EL表达式在数据输出渲染时比JSP的表达式简洁很多。

**四大域**

1. pageContext
2. request
3. session
4. application

作用域逐渐增大，${key}在查询时也是由小到大查找。

**其他内置对象**

- pageContext：获取JSP的九大内置对象
- param：获取请求参数
- paramValues：获取请求参数的数组形式
- header：获取请求头信息
- headerValues：获取请求头信息的数组形式
- cookie：获取cookie信息
- initParam：获取web.xml中配置的\<context-param>上下文参数

### JSTL

JSTL用于替代JSP的代码指令，诸如循环、判断、遍历等逻辑语句。

**五组标签库**

需要导包并使用taglib引入

![image-20210330105454903](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210330105454903.png)