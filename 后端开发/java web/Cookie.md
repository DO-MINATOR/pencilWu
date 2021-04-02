### Cookie简介

- 服务器通知客户端保存形式为键值对的对象
- 客户端每次请求都会附送cookie给服务器（默认web工程路径）
- 单个cookie不能超过4kb

### 创建和添加

```java
Cookie cookie = new Cookie("key","value");
resp.addCookie(cookie);
//必须要add，一次请求可以添加多个cookie
```

### 服务器获取

```java
Cookie[] cookies = req.getCookies();
for(Cookie cookie : cookies){
	//...
}
```

### 设置存活时间

```java
cookie.setAge(int);
```

1. \>0存活秒数
2. ==0立即删除
3. <0(默认)推出浏览器后删除

### 设置path

通过path设置哪些cookie可以发送到服务器

![image-20210402114335492](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210402114335492.png)