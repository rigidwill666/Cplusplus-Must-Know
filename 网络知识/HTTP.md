# HTTP方法
| 方法 | 说明 |
| --- | ----------- |
| GET	| 用于请求获取资源，不会对服务器上的资源造成影响（即**幂等**）。例如获取网页、图片等|
| PUT	| 用于更新资源（整体更新），如果资源不存在可以创建。也是**幂等**的|
| DELETE	| 用于请求删除指定资源。也是**幂等**的|
| POST	| 用于提交数据给服务器，例如表单提交、上传文件。服务器可能会创建新的资源|
| PATCH	|对资源进行部分修改，区别于PUT的“整体更新”|
| HEAD | 类似于GET请求，但服务器响应中只返回头部，不返回实际内容。常用于检测资源是否存在、获取元数据|
| OPTIONS	| 查询服务器支持哪些HTTP方法，通常用于跨域请求中的预检请求（preflight）|


> 幂等性？：
>
> 多次发送请求的效果一样。例如：多次DELETE一个已经不存在的资源，结果都一样。
>
> 安全性？：
>
> 不会修改服务器资源的操作是“安全”的，例如GET和HEAD。
# HTTP报文
1. 起始行
- 请求报文：请求方法 请求URL HTTP版本

  示例：GET /index.html HTTP/1.1
- 响应报文：HTTP版本 状态码 状态描述

  示例：HTTP/1.1 200 OK
2. 首部字段

    用于描述客户端或服务器的环境信息（元数据），每行是**键: 值**格式。
- 常见请求头字段（客户端→服务器）：

  Host: 请求的主机地址（必须有）

  User-Agent: 浏览器或客户端信息

  Accept: 可接受的内容类型（如 text/html, application/json）

  Content-Type: 请求体的类型（如 POST 提交的表单）

  Authorization: 认证信息

- 常见响应头字段（服务器→客户端）：

  Content-Type: 响应内容的类型

  Content-Length: 响应体长度（字节数）

  Set-Cookie: 设置cookie

  Server: 服务器软件信息
3. 空行

    起始行和首部字段后会有一个空行，用来分隔**头部和正文**。格式上是 \r\n（回车换行）。
4. 消息体【可选】

    请求或响应中实际要传输的数据内容。

    在 GET 请求中一般为空；POST、PUT 常常有消息体（如JSON、表单数据等）。

    响应体中一般是 HTML、JSON、文件等内容。
## 请求报文
~~~
GET /hello.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html
~~~
## 响应报文
~~~
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 125

<html>
  <body>
    <h1>Hello, world!</h1>
  </body>
</html>
~~~
