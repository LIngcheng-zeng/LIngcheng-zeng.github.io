[toc]

# URL

用户在为获取资源，需要了解和执行的动作太多。不同的资源，执行的动作也不同。
URL 统一了用户访问获取资源的方式。

URL 常用格式如下

```html
<scheme>://<host>:<port>/<path>;<param>?<query>
<scheme>://<user>:<password>@<host>:<port>/<path>;<param>?<query>

<path> 路径段可以有多个，每个<path> 都可以有自己的<param>
<param> 的格式：key=value, 存在多个时用 ； 分割
<query> 的格式：key=value，存在多个时 用 & 分割
```

## URL的 可移植性 和完整性

URL 需要在各种 因特网协议 中 安全的，完整的传输，同时保证人类可读，不能有不可见字符。
### 字符集
ASCII的可见非受限字符 + 不安全的字符转义机制。
转义表示法：% 后面跟着两个ASCII码的16进制数。

发送URL前，对不安全的字符进行编码，在应用程序解释URL前，要对URL进行解码。

# HTTP
请求报文结构
```html
<method> <request-url> <version>
<headers>

<entity-body>

header 可以有 0个 或者多个。

每行一个，单行格式 key : value。 

首部以空行结束。

HTTP规范定义了几种首部字段，应用程序可以随意发明自己的首部字段。

```

响应报文结构
```html
<version> <status> <reason-phrase>
<headers>

<entity-body>
```



首部和方法配合工作，共同决定了客户端和服务器能够做什么事情。

# 
