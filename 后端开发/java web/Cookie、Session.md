### Cookie

- 服务器通知客户端保存形式为键值对的对象
- 客户端每次请求都会附送cookie给服务器（默认web工程路径）
- 单个cookie不能超过4kb

#### 创建和添加

```java
Cookie cookie = new Cookie("key","value");
resp.addCookie(cookie);
//必须要add，一次请求可以添加多个cookie
```

#### 服务器获取

```java
Cookie[] cookies = req.getCookies();
for(Cookie cookie : cookies){
	//...
}
```

#### 设置存活时间

```java
cookie.setAge(int);
```

1. \>0存活秒数
2. ==0立即删除
3. <0(默认)推出浏览器后删除

#### 设置path

通过path设置哪些cookie可以发送到服务器

![image-20210402114335492](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210402114335492.png)

### Session

- Session是一个接口
- 保存在服务器，用于维护客户端和服务器间的关联
- 每个客户端都有自己的session会话

#### 判断是否新创建的session

```java
Session session = req.getSession();
session.isNew();
session.getId();//获取唯一id
```

#### 设置和读取域信息

```java
session.setAttribute("key","value");
session.getAttribute("key");
```

#### 生命周期控制

```java
session.setMaxInactiveInterval();
session.getMaxInactiveInterval();
```

可以在私有的web工程下的web.xml中修改默认值。

**注意：**超时时长指的是客户端两次请求间的间隔时长。

![image-20210403100703201](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210403100703201.png)

#### 立即销毁

```java
session.invalidate();
```

#### session获取底层原理

当服务器设置session以后，相应的会给客户端发送一个获取该session的cookie，这样二次请求时，就可以根据此cookie获取到session对象。

![image-20210405091450588](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210405091450588.png)