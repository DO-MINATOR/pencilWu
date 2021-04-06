### 简介

Ajax即"Asynchronous Javascript And XML"(异步JavaScript和XML)，异步局部请求，局部更新页面。地址栏不会发生变化，也不会舍弃原页面的内容。

### 步骤

1. js创建xmlhttprequest对象
2. 调用open()设置请求方法、请求路径，是否异步
3. 设置状态响应事件
4. 调用send()发送请求

readyState存有XMLHttpRequest的状态，分别是：

- 0：请求初始化
- 1：服务器建立连接
- 2：请求已接收
- 3：请求处理中
- 4：请求完成，且响应就绪

### JQuery中的Ajax

JQuery中的Ajax替代了原生的js的Ajax请求。参数如下：

- url：请求地址
- type：请求类型
- data：发送给服务器的数据
- success：请求且响应成功后的回调函数
- dataType：响应的数据类型（text、json）

示例

```javascript
$("#ajaxBtn").click(function(){
    $.ajax({
        url:"http://127.0.0.1:8080/json/ajaxservlet",
        data:{action:"jQueryAjax"},
        type:"GET",
        success:function(data){
            $("#msg").html("编号:"+data.id+" ，姓名:"+data.name);
        },
        dataType:"json"
    })    
})
```

