### File Upload

#### html

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="" method="post" enctype="multipart/form-data">
        用户名：<input type="text" name="username"><br>
        头像：<input type="file" name="photo"><br>
        <input type="submit" value="上传">
    </form>
</body>
</html>
```

- post请求提供更大的上传空间和安全保护
- enctype设置multipart/form-data将请求体的多段数据合并并转换为二进制流数据进行传输
- input type设置为file

#### Servlet

在doPost通过request获取到inputstream读取流数据。

### File Download

#### Servlet

1. 设置响应头中的数据类型MimeType
3. 设置响应头中的接受动作（是否下载）
4. 读取服务器文件
5. 通过servletContext获取输入流
6. 通过第三方jar包将输入流数据复制到response的输出流中

**注意：**使用URLEncoder.encode将中文进行url编码，防止下载中文名文件乱码，有的浏览器需要base64编解码。

