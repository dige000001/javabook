## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# URL和URI

与 URI（统一资源标识符）相比，我们更熟悉 URL（Uniform Resource Locator，统一资源定位符）。URL 正是使用 Web 浏览器等访问 Web 页面时需要输入的网页地址。比如，`http://byr.pt/` 就是 URL。 

URI 用字符串标识某一互联网资源，而 URL 表示资源的地点（互联网上所处的位置）。可见 URL 是 URI 的子集。 

## URI 格式 

绝对 URI 的格式：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8030ee69345343cea6991e5b0f463bec~tplv-k3u1fbpfcp-zoom-1.image)

但并不是所有的应用程序都符合RFC标准，开发者有可能自定标准

# HTTP请求和响应

## 请求报文

请求报文是由请求方法、请求 URI、协议版本、可选的请求首部字 段和内容实体构成的。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9a86d4567204459b25a05acfbb307ef~tplv-k3u1fbpfcp-zoom-1.image)

## 响应报文

响应报文基本上由协议版本、状态码（表示请求成功或失败的数字 代码）、用以解释状态码的原因短语、可选的响应首部字段以及实体主体构成。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1dd71a9ef3c4aecbcb62a8a57e2279f~tplv-k3u1fbpfcp-zoom-1.image)

## GET

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b7679b8f4d345c9b72459036321b36d~tplv-k3u1fbpfcp-zoom-1.image)

## POST(一般用来发送数据)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcadff10b8b64fdd9abce5e910aaac59~tplv-k3u1fbpfcp-zoom-1.image)

## PUT：传输文件 

PUT 方法用来传输文件。就像 FTP 协议的文件上传一样，要求在 请求报文的主体中包含文件内容，然后保存到请求 URI 指定的位置。 

但是，鉴于 HTTP/1.1 的 PUT 方法自身不带验证机制，任何人都可 以上传文件 , 存在安全性问题，因此一般的 Web 网站不使用该方法。若 配合 Web 应用程序的验证机制，或架构设计采用 REST（REpresentational State Transfer，表征状态转移）标准的同类 Web 网站，就可能会开放使 用 PUT 方法。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fce4b95c33f4661a173e4a265aefb97~tplv-k3u1fbpfcp-zoom-1.image)

## DELETE：删除文件 

DELETE 方法用来删除文件，是与 PUT 相反的方法。DELETE 方 法按请求 URI 删除指定的资源。 但是，HTTP/1.1 的 DELETE 方法本身和 PUT 方法一样不带验证机 制，所以一般的 Web 网站也不使用 DELETE 方法。当配合 Web 应用程 序的验证机制，或遵守 REST 标准时还是有可能会开放使用的。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24efc617be85431b9f58bef899417d6d~tplv-k3u1fbpfcp-zoom-1.image)

## HEAD：获得报文首部 

HEAD 方法和 GET 方法一样，只是不返回报文主体部分。用于确认 URI 的有效性及资源更新的日期时间等 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/225466ad9d8047d9ae9cd159f262ca92~tplv-k3u1fbpfcp-zoom-1.image)

## OPTIONS：询问支持的方法 

OPTIONS 方法用来查询针对请求 URI 指定的资源支持的方法。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71ae3147e39f496190bbe12454e56183~tplv-k3u1fbpfcp-zoom-1.image)

## TRACE：追踪路径 

TRACE 方法是让 Web 服务器端将之前的请求通信环回给客户端的 方法。 发送请求时，在 Max-Forwards 首部字段中填入数值，每经过一个 服务器端就将该数字减 1，当数值刚好减到 0 时，就停止继续传输，最后接收到请求的服务器端则返回状态码 200 OK 的响应。 

客户端通过 TRACE 方法可以查询发送出去的请求是怎样被加工修 改 / 篡改的。这是因为，请求想要连接到源目标服务器可能会通过代理中转，TRACE 方法就是用来确认连接过程中发生的一系列操作。 

但是，TRACE 方法本来就不怎么常用，再加上它容易引发 XST （Cross-Site Tracing，跨站追踪）攻击，通常就更不会用到了。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e35b3176b53b4c4a918d5edeab762d46~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a45bedd9ffda45efb5509d7d09c7d611~tplv-k3u1fbpfcp-zoom-1.image)

## CONNECT：要求用隧道协议连接代理 

CONNECT 方法要求在与代理服务器通信时建立隧道，实现用隧道 协议进行 TCP 通信。主要使用 SSL（Secure Sockets Layer，安全套接 层）和 TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输。 CONNECT 方法的格式如下所示。 

`CONNECT 代理服务器名:端口号 HTTP版本 `

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2b2fccc49094a24ae44f207c2e13a73~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85a1984d5c7646df8b939b6a77f72b58~tplv-k3u1fbpfcp-zoom-1.image)

# 降低通信开销，加快页面响应

## 可持久化连接

使用浏览器浏览一个包含多张图片的 HTML 页面时，在发送请求访问 HTML 页面资源的同时，也会请求该 HTML 页面里包含的其他资源。早期对每个资源的请求都有TCP连接建立和断开，因此，每次的请求都会造成无谓的 TCP 连接建立和断开， 增加通信量的开销。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83ad6554f37b461098a28c3c3ecce22d~tplv-k3u1fbpfcp-zoom-1.image)


为解决上述 TCP 连接的问题，HTTP/1.1 和一部分的 HTTP/1.0 想出 了持久连接（HTTP Persistent Connections，也称为 HTTP keep-alive 或 HTTP connection reuse）的方法。持久连接的特点是，只要任意一端没 有明确提出断开连接，则保持 TCP 连接状态。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49208964306c44509ef138cc890b75bd~tplv-k3u1fbpfcp-zoom-1.image)

在 HTTP/1.1 中，所有的连接默认都是持久连接 

## 管道化

持久连接使得多数请求以管线化（pipelining）方式发送成为可能。 从前发送请求后需等待并收到响应，才能发送下一个请求。管线化技术 出现后，不用等待响应亦可直接发送下一个请求。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08f695c23bba4c43a869d090108388d8~tplv-k3u1fbpfcp-zoom-1.image)

# 使用 Cookie 的状态管理 

HTTP 是无状态协议，它不对之前发生过的请求和响应的状态进行管理。也就是说，无法根据之前的状态进行本次的请求处理，无状态的好处是减少了通信开销。 但假设要求登录认证的 Web 页面本身无法进行状态的管理（不记录已登录的状态），那么每次跳转新页面不是要再次登录，就是要在每次请求报文中附加参数来管理登录状态。

保留无状态协议这个特征的同时又要解决类似的矛盾问题，于是引 入了 Cookie 技术。Cookie 技术通过在请求和响应报文中写入 Cookie 信 息来控制客户端的状态。 

Cookie 会根据从服务器端发送的响应报文内的一个叫做 Set-Cookie 的首部字段信息，通知客户端保存 Cookie。当下次客户端再往该服务器发送请求时，客户端会自动在请求报文中加入 Cookie 值后发送出去。 服务器端发现客户端发送过来的 Cookie 后，会去检查究竟是从哪 一个客户端发来的连接请求，然后对比服务器上的记录，最后得到之前 的状态信息 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76fd9c5f5b7b476eb99f1c22ac1b7037~tplv-k3u1fbpfcp-zoom-1.image)

发生 Cookie 交互的情景，HTTP 请求报文和响应报文的 内容如下 ：\
1 请求报文（没有 Cookie 信息的状态）\


Java复制代码

```
```

1

```
GET /reader/ HTTP/1.1
```

2

```
Host: hackr.jp
```

3

```
*首部字段内没有Cookie的相关信息
```

2响应报文 （服务器端生成 Cookie 信息）\


Java复制代码

```
```

1

```
HTTP/1.1 200 OK
```

2

```
Date: Thu, 12 Jul 2012 07:12:20 GMT
```

3

```
Server: Apache
```

4

```
＜Set-Cookie: sid=1342077140226724; path=/; expires=Wed,10-Oct-12 07:12:20 GMT＞
```

5

```
Content-Type: text/plain; charset=UTF-8
```

3 请求报文（自动发送保存着的 Cookie 信息）


Java复制代码

```
```

1

```
GET /image/ HTTP/1.1
```

2

```
Host: hackr.jp
```

3

```
Cookie: sid=1342077140226724
```