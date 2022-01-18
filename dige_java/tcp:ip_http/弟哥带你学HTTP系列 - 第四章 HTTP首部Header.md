## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# HTTP 报文首部 

## HTTP 请求报文 

在请求中，HTTP 报文由方法、URI、HTTP 版本、HTTP 首部字段 等部分构成。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7cea5dacd7ae448b9ce2a533de2ef735~tplv-k3u1fbpfcp-zoom-1.image)

## HTTP 响应报文 

在响应中，HTTP 报文由 HTTP 版本、状态码（数字和原因短语）、 HTTP 首部字段 3 部分构成。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9b5219870ff4082b2302ae3b94089d3~tplv-k3u1fbpfcp-zoom-1.image)

# HTTP首部字段

Q: 若 HTTP 首部字段重复了会如何 

A: 当 HTTP 报文首部中出现了两个或两个以上具有相同首部字段名时会 怎么样？这种情况在规范内尚未明确，根据浏览器内部处理逻辑的不同， 结果可能并不一致。有些浏览器会优先处理第一次出现的首部字段，而有 些则会优先处理最后出现的首部字段。 

## 4 种 HTTP 首部字段类型 

-   **通用首部字段**（General Header Fields）：请求报文和响应报文两方都会使用的首部。 
-   **请求首部字段**（Request Header Fields） 从客户端向服务器端发送请求报文时使用的首部。补充了请求的附 加内容、客户端信息、响应内容相关优先级等信息。 

<!---->

-   **响应首部字段**（Response Header Fields） 从服务器端向客户端返回响应报文时使用的首部。补充了响应的附 加内容，也会要求客户端附加额外的内容信息。 
-   **实体首部字段**（Entity Header Fields） 针对请求报文和响应报文的实体部分使用的首部。补充了资源内容 更新时间等与实体有关的信息。 

## HTTP/1.1 首部字段一览 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62b2e3457abd4bd6b067292adf02510e~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/575cdb55bdc54a9ebe7239880eae532c~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/867f41bfc39f460191dfeb62d8be5c4d~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31311bf33ca34a888a450cad2a595600~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1eea2fbdce214249b5b104df16ec2d0e~tplv-k3u1fbpfcp-zoom-1.image)




## End-to-end 首部和 Hop-by-hop 首部 

HTTP 首部字段将定义成缓存代理和非缓存代理的行为，分成 2 种 类型。 

-   **端到端首部**（End-to-end Header） 分在此类别中的首部会转发给请求 / 响应对应的最终接收目标，且 必须保存在由缓存生成的响应中，另外规定它必须被转发。 
-   **逐跳首部**（Hop-by-hop Header） 分在此类别中的首部只对单次转发有效，会因通过缓存或代理而不 再转发。HTTP/1.1 和之后版本中，如果要使用 hop-by-hop 首部， 需提供 Connection 首部字段。 

下面列举了 HTTP/1.1 中的逐跳首部字段。除这 8 个首部字段之外， 其他所有字段都属于端到端首部 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7cc6d93559bd412b8d499b6751a8aa43~tplv-k3u1fbpfcp-zoom-1.image)

# HTTP/1.1 通用首部字段 

通用首部字段是指，请求报文和响应报文双方都会使用的首部。 

## Cache-Control 

通过指定首部字段 Cache-Control 的指令，就能操作缓存的工作机制。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ec4bcb064d7481bb7ce3392fbb25b89~tplv-k3u1fbpfcp-zoom-1.image)

指令的参数是可选的，多个指令之间通过“,”分隔。首部字段 Cache-Control 的指令可用于请求及响应时。 如：`Cache-Control: private, max-age=0, no-cache `

可用的指令按请求和响应分类如下所示。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dac62edb2fa246c8a9b59dc2338f5db9~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c149019cb41435a974fcf2c034eed69~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5efd67fba1d44b29a2bdb037ab55f78d~tplv-k3u1fbpfcp-zoom-1.image)

### private指令 

当指定 private 指令后，响应只以特定的用户作为对象，这与 public 指令的行为相反。 

缓存服务器会对该特定用户提供资源缓存的服务，对于其他用户发 送过来的请求，代理服务器则不会返回缓存。 

### no-cache指令 

使用 no-cache 指令的目的是为了防止从缓存中返回过期的资源。 客户端发送的请求中如果包含 no-cache 指令，**则表示客户端将不会 接收缓存过的响应。** 于是，“中间”的缓存服务器必须把客户端请求转发给源服务器。 如果服务器返回的响应中包含 no-cache 指令，**那么缓存服务器不能对资源进行缓存。源服务器以后也将不再对缓存服务器请求中提出的资源有效性进行确认，且禁止其对响应资源进行缓存操作。**

服务器返回的响应中，若报文首部字段 Cache-Control 中对 no-cache 字段名具体指定参数值，那么客户端在接收到这个被指定参数值 的首部字段对应的响应报文后，就不能使用缓存。换言之，无参数值的 首部字段可以使用缓存。只能在响应指令中指定该参数。 如：` Cache-Control: no-cache=Location `

### no-store指令 

当使用 no-store 指令 A 时，暗示请求（和对应的响应）或响应中包含机密信息。 因此，该指令规定缓存不能在本地存储请求或响应的任一部分。 

从字面意思上很容易把no-cache误解成为不缓存，但事实上no-cache代表不缓存过期的资源，缓存会向源服务器进行有效期确认后处理资源，也许称为 do-not-serve-from-cache-without-revalidation更合适。no-store才是真正地不进 行缓存 

### s-maxage指令 

s-maxage 指令的功能和 max-age 指令的相同，它们的不同点是：s-maxage 指令只适用于供多位用户使用的公共缓存服务器 A 。也就是说， 对于向同一用户重复返回响应的服务器来说，这个指令没有任何作用。

另外，当使用 s-maxage 指令后，则直接忽略对 Expires 首部字段及 max-age 指令的处理。 

### min-fresh指令 

min-fresh 指令要求缓存服务器返回至少还未过指定时间的缓存资源。 比如，当指定 min-fresh 为 60 秒后，过了 60 秒的资源都无法作为 响应返回了 

### max-stale指令 

使用 max-stale 可指示缓存资源，即使过期也照常接收。 如果指令未指定参数值，那么无论经过多久，客户端都会接收响应； 如果指令中指定了具体数值，那么即使过期，只要仍处于 max-stale 指定的时间内，仍旧会被客户端接收。 

### only-if-cached指令 

使用 only-if-cached 指令表示客户端仅在缓存服务器本地缓存有目标资源的情况下才会要求其返回。换言之，该指令要求缓存服务器不重新加载响应，也不会再次确认资源有效性。若发生请求缓存服务器的本地缓存无响应，则返回状态码 504 Gateway Timeout。 

### must-revalidate指令 

使用 must-revalidate 指令，代理会向源服务器再次验证即将返回的 响应缓存目前是否仍然有效。 若代理无法连通源服务器再次获取有效资源的话，缓存必须给客户 端一条 504（Gateway Timeout）状态码。 

另外，使用 must-revalidate 指令会忽略请求的 max-stale 指令（即使 已经在首部使用了 max-stale，也不会再有效果）。 

### proxy-revalidate指令 

proxy-revalidate 指令要求所有的缓存服务器在接收到客户端带有该 指令的请求返回响应之前，必须再次验证缓存的有效性。 

### no-transform指令 

使用 no-transform 指令规定无论是在请求还是响应中，缓存都不能改变实体主体的媒体类型。 这样做可防止缓存或代理压缩图片等类似操作。 

## Connection 

Connection 首部字段具备如下两个作用。 

● 控制不再转发给代理的首部字段

● 管理持久连接 

### 控制不再转发的首部字段 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4aec2021f349470492957f319d6884e3~tplv-k3u1fbpfcp-zoom-1.image)

### 管理持久连接 

HTTP/1.1 版本的默认连接都是持久连接。为此，客户端会在持久连接上连续发送请求。当服务器端想明确断开连接时，则指 定 Connection 首部字段的值为 Close。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf637e147e75450390edefe15bf64d7a~tplv-k3u1fbpfcp-zoom-1.image)

### Trailer 

首部字段 Trailer 会事先说明在报文主体后记录了哪些首部字段。该首部字段可应用在 HTTP/1.1 版本分块传输编码时。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a897bf60425f43a1984a36800db92cb4~tplv-k3u1fbpfcp-zoom-1.image)

```
HTTP/1.1 200 OK
Date: Tue, 03 Jul 2012 04:40:56 GMT
Content-Type: text/html
...
Transfer-Encoding: chunked
Trailer: Expires
...(报文主体)...
0
Expires: Tue, 28 Sep 2004 23:59:59 GMT
```

以上用例中，指定首部字段 Trailer 的值为 Expires，在报文主体之 后（分块长度 0 之后）出现了首部字段 Expires。 

### Transfer-Encoding 

首部字段 Transfer-Encoding 规定了传输报文主体时采用的编码 方式。 

HTTP/1.1 的传输编码方式仅对分块传输编码有效。 

### Upgrade 

首部字段 Upgrade 用于检测 HTTP 协议及其他协议是否可使用更高 的版本进行通信，其参数值可以用来指定一个完全不同的通信协议。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d59417624c1a4244bcb49296b6afb1ac~tplv-k3u1fbpfcp-zoom-1.image)

上图用例中，首部字段 Upgrade 指定的值为 TLS/1.0。请注意此处 两个字段首部字段的对应关系，Connection 的值被指定为 Upgrade。 Upgrade 首部字段产生作用的 Upgrade 对象仅限于客户端和邻接服务器之间。因此，使用首部字段 Upgrade 时，还需要额外指定 Connection: Upgrade。 

对于附有首部字段 Upgrade 的请求，服务器可用 101 Switching Protocols 状态码作为响应返回。 

### Via 

使用首部字段 Via 是为了追踪客户端与服务器之间的请求和响应报 文的传输路径。 报文经过代理或网关时，会先在首部字段 Via 中附加该服务器的信 息，然后再进行转发。这个做法和 traceroute 及电子邮件的 Received 首 部的工作机制很类似。 

首部字段 Via 不仅用于追踪报文的转发，还可避免请求回环的发生。所以必须在经过代理时附加该首部字段内容。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a565f22d2fbb49b4be3db04678fe362f~tplv-k3u1fbpfcp-zoom-1.image)Via 首部是为了追踪传输路径，所以经常会和 TRACE 方法一起使 用。比如，代理服务器接收到由 TRACE 方法发送过来的请求（其中 Max-Forwards: 0）时，代理服务器就不能再转发该请求了。这种情 况下，代理服务器会将自身的信息附加到 Via 首部后，返回该请求的 响应 

### Warning 

HTTP/1.1 的 Warning 首部是从 HTTP/1.0 的响应首部（Retry-After） 演变过来的。该首部通常会告知用户一些与缓存相关的问题的警告。

Warning 首部的格式如下。最后的日期时间部分可省略。 

`Warning: [警告码][警告的主机:端口号]“[警告内容]”([日期时间]) `

如： 

`Warning: 113 gw.hackr.jp:8080 "Heuristic expiration" Tue, 03 Jul 2012 05:09:44 GMT `

### Pragma 

Pragma 是 HTTP/1.1 之前版本的历史遗留字段，仅作为与 HTTP/1.0 的向后兼容而定义。 规范定义的形式唯一：` Pragma: no-cache `

该首部字段属于通用首部字段，但只用在客户端发送的请求中。客 户端会要求所有的中间服务器不返回缓存的资源 

所有的中间服务器如果都能以 HTTP/1.1 为基准，那直接采用 Cache-Control: no-cache 指定缓存的处理方式是最为理想的。但要整体 掌握全部中间服务器使用的 HTTP 协议版本却是不现实的。因此，发送 的请求会同时含有下面两个首部字段。 

### Data

说明报文发送时间

# 请求首部字段 

请求首部字段是从客户端往服务器端发送请求报文中所使用的字 段，用于补充请求的附加信息、客户端信息、对响应内容相关的优先级 等内容。 

## Accept 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc9d7a863e3246a4870b4ac6a2243bf0~tplv-k3u1fbpfcp-zoom-1.image)

Accept 首部字段可通知服务器，用户代理能够处理的媒体类型及媒 体类型的相对优先级。 可使用 type/subtype 这种形式，一次指定多种媒体类型。 

-   文本文件 text/html, text/plain, text/css ... application/xhtml+xml, application/xml ... 
-   图片文件 image/jpeg, image/gif, image/png ... 

<!---->

-   视频文件 video/mpeg, video/quicktime ... 
-   应用程序使用的二进制文件 application/octet-stream, application/zip ... 

若想要给显示的媒体类型增加优先级，则使用 q= 来额外表示权重 值 A ，用分号（;）进行分隔。权重值 q 的范围是 0~1（可精确到小数点后 3 位），且 1 为最大值。不指定权重 q 值时，默认权重为 q=1.0。 当服务器提供多种内容时，将会首先返回权重值最高的媒体类型。 

## Accept-Charset 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/193c8730530d47a9ac84a0c877db9a25~tplv-k3u1fbpfcp-zoom-1.image)

Accept-Charset 首部字段可用来通知服务器用户代理支持的字符集 及字符集的相对优先顺序。另外，可一次性指定多种字符集。与首部字 段 Accept 相同的是可用权重 q 值来表示相对优先级。 

该首部字段应用于内容协商机制的服务器驱动协商。 

## Accept-Encoding 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc3d94837a7b44b2be932242fa4d3ac8~tplv-k3u1fbpfcp-zoom-1.image)

Accept-Encoding 首部字段用来告知服务器用户代理支持的内容编 码及内容编码的优先级顺序。可一次性指定多种内容编码。 

## Accept-Language 

首部字段 Accept-Language 用来告知服务器用户代理能够处理的自 然语言集（指中文或英文等），以及自然语言集的相对优先级。可一次 指定多种自然语言集。 和 Accept 首部字段一样，按权重值 q 来表示相对优先级。

## Authorization 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28628d908a9642819390d10cbfa9c028~tplv-k3u1fbpfcp-zoom-1.image)

## Expect 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed258cc49f2248ca8ba5fe4a224a82c3~tplv-k3u1fbpfcp-zoom-1.image)

客户端使用首部字段 Expect 来告知服务器，期望出现的某种特定 行为。因服务器无法理解客户端的期望作出回应而发生错误时，会返回 状态码 417 Expectation Failed。 

客户端可以利用该首部字段，写明所期望的扩展。虽然 HTTP/1.1 规范只定义了 100-continue（状态码 100 Continue 之意）。 

等待状态码 100 响应的客户端在发生请求时，需要指定 Expect: 100-continue。 

## From 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b04256942ab40a8ae586ee8c1f41970~tplv-k3u1fbpfcp-zoom-1.image)

首部字段 From 用来告知服务器使用用户代理的用户的电子邮件地 址。 通常，其使用目的就是为了显示搜索引擎等用户代理的负责人的 电子邮件联系方式。使用代理时，应尽可能包含 From 首部字段（但可 能会因代理不同，将电子邮件地址记录在 User-Agent 首部字段内）。 

## Host 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf6cf656232849cfab5436a81f40ab14~tplv-k3u1fbpfcp-zoom-1.image)

首部字段 Host 会告知服务器，请求的资源所处的互联网主机名和 端口号。Host 首部字段在 HTTP/1.1 规范内是唯一一个必须被包含在请 求内的首部字段。 

首部字段 Host 和以单台服务器分配多个域名的虚拟主机的工作机 制有很密切的关联，这是首部字段 Host 必须存在的意义。 

请求被发送至服务器时，请求中的主机名会用 IP 地址直接替换解 决。但如果这时，相同的 IP 地址下部署运行着多个域名，那么服务器 就会无法理解究竟是哪个域名对应的请求。因此，就需要使用首部字段 Host 来明确指出请求的主机名。若服务器未设定主机名，那直接发送一个空值即可。

## If-Match 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/336929c953684dc0ae0e0ffe64a97269~tplv-k3u1fbpfcp-zoom-1.image)

## If-Modified-Since 

首部字段 If-Modified-Since，属附带条件之一，它会告知服务器若 If-Modified-Since 字段值早于资源的更新时间，则希望能处理该请求。 

而在指定 If-Modified-Since 字段值的日期时间之后，如果请求的资源都 没有过更新，则返回状态码 304 Not Modified 的响应。

If-Modified-Since 用于确认代理或客户端拥有的本地资源的有效 性。获取资源的更新日期时间，可通过确认首部字段 Last-Modified 来 确定。 

## If-None-Match 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/399c32ed58344a74a9e4c0a22442288c~tplv-k3u1fbpfcp-zoom-1.image)

首部字段 If-None-Match 属于附带条件之一。它和首部字段 IfMatch 作用相反。用于指定 If-None-Match 字段值的实体标记（ETag） 值与请求资源的 ETag 不一致时，它就告知服务器处理该请求。 

在 GET 或 HEAD 方法中使用首部字段 If-None-Match 可获取最新 的资源。因此，这与使用首部字段 If-Modified-Since 时有些类似。 

## If-Range 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4ff43e36adc408386130b1287647ec5~tplv-k3u1fbpfcp-zoom-1.image)

## If-Unmodified-Since 

不用说

## Max-Forwards 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd155694e4d04a6f855d86d7c5358b15~tplv-k3u1fbpfcp-zoom-1.image)

通 过 TRACE 方 法 或 OPTIONS 方 法， 发 送 包 含 首 部 字 段 MaxForwards 的请求时，该字段以十进制整数形式指定可经过的服务器最大 数目。服务器在往下一个服务器转发请求之前，Max-Forwards 的值减 1 后重新赋值。当服务器接收到 Max-Forwards 值为 0 的请求时，则不 再进行转发，而是直接返回响应。 

使用 HTTP 协议通信时，请求可能会经过代理等多台服务器。途 中，如果代理服务器由于某些原因导致请求转发失败，客户端也就等不 到服务器返回的响应了。对此，我们无从可知。

可以灵活使用首部字段 Max-Forwards，针对以上问题产生的原因展开 调查。由于当 Max-Forwards 字段值为 0 时，服务器就会立即返回响应，由 此我们至少可以对以那台服务器为终点的传输路径的通信状况有所把握。 

## Proxy-Authorization 

接收到从代理服务器发来的认证质询时，客户端会发送包含首部字 段 Proxy-Authorization 的请求，以告知服务器认证所需要的信息。 这个行为是与客户端和服务器之间的 HTTP 访问认证相类似的，不 同之处在于，认证行为发生在客户端与代理之间。客户端与服务器之间 的认证，使用首部字段 Authorization 可起到相同作用。 

## Range 

接收到附带 Range 首部字段请求的服务器，会在处理请求之后返回 状态码为 206 Partial Content 的响应。无法处理该范围请求时，则会返 回状态码 200 OK 的响应及全部资源。 

## User-Agent 

首部字段 User-Agent 会将创建请求的浏览器和用户代理名称等信息 传达给服务器。 

由网络爬虫发起请求时，有可能会在字段内添加爬虫作者的电子邮 件地址。此外，如果请求经过代理，那么中间也很可能被添加上代理服 务器的名称 

# 响应首部字段 

## Accept-Ranges 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/011b9f99308c4c58a9eda66339b83b2d~tplv-k3u1fbpfcp-zoom-1.image)

首部字段 Accept-Ranges 是用来告知客户端服务器是否能处理范围 请求，以指定获取服务器端某个部分的资源。 

可指定的字段值有两种，可处理范围请求时指定其为 bytes，反之 则指定其为 none。 

## Age 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/377bea0bd58a41ae83d732ce6ca4f169~tplv-k3u1fbpfcp-zoom-1.image)

首部字段 Age 能告知客户端，源服务器在多久前创建了响应。字段 值的单位为秒。 若创建该响应的服务器是缓存服务器，Age 值是指缓存后的响应再次 发起认证到认证完成的时间值。

代理创建响应时必须加上首部字段 Age。 

## ETag 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b941b8838c94d06ae7c043e43e5f807~tplv-k3u1fbpfcp-zoom-1.image)

首部字段 ETag 能告知客户端实体标识。它是一种可将资源以字符 串形式做唯一性标识的方式。服务器会为每份资源分配对应的 ETag 值。 另外，当资源更新时，ETag 值也需要更新。生成 ETag 值时，并没 有统一的算法规则，而仅仅是由服务器来分配。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a691fe3e872343c38087cd7c9f970e7b~tplv-k3u1fbpfcp-zoom-1.image)

资源被缓存时，就会被分配唯一性标识。例如，当使用中文版的浏 览器访问 http ：//www.google.com/ 时，就会返回中文版对应的资源，而 使用英文版的浏览器访问时，则会返回英文版对应的资源。两者的 URI 是相同的，所以仅凭 URI 指定缓存的资源是相当困难的。若在下载过 程中出现连接中断、再连接的情况，都会依照 ETag 值来指定资源。 

ETag 中有强 ETag 值和弱 ETag 值之分： 

-   强 ETag 值不论实体发生多么细微的变化都会改变其值。 
-   弱 ETag 值只用于提示资源是否相同。只有资源发生了根本改变，产 生差异时才会改变 ETag 值。这时，会在字段值最开始处附加 W/ 




## Location 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b35759e0fa647ebb5373006c03fa485~tplv-k3u1fbpfcp-zoom-1.image)

使用首部字段 Location 可以将响应接收方引导至某个与请求 URI 位置不同的资源。

基本上，该字段会配合 3xx ：Redirection 的响应，提供重定向的 URI。 

几乎所有的浏览器在接收到包含首部字段 Location 的响应后，都会强制性地尝试对已提示的重定向资源的访问 

## Proxy-Authenticate 

首部字段 Proxy-Authenticate 会把由代理服务器所要求的认证信息 发送给客户端。 它与客户端和服务器之间的 HTTP 访问认证的行为相似，不同之处 在于其认证行为是在客户端与代理之间进行的。而客户端与服务器之间 进行认证时，首部字段 WWW-Authorization 有着相同的作用。 

## Retry-After 

首部字段 Retry-After 告知客户端应该在多久之后再次发送请求。主 要配合状态码 503 Service Unavailable 响应，或 3xx Redirect 响应一起 使用。 

字段值可以指定为具体的日期时间（Wed, 04 Jul 2012 06：34：24 GMT 等格式），也可以是创建响应后的秒数。 

## Server 

首部字段 Server 告知客户端当前服务器上安装的 HTTP 服务器应用 程序的信息。不单单会标出服务器上的软件应用名称，还有可能包括版 本号和安装时启用的可选项。 

## Vary 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c01c94be9272418b9a22d03364b841b0~tplv-k3u1fbpfcp-zoom-1.image)

首部字段 Vary 可对缓存进行控制。源服务器会向代理服务器传达 关于本地缓存使用方法的命令。 

从代理服务器接收到源服务器返回包含 Vary 指定项的响应之后， 若再要进行缓存，仅对请求中含有相同 Vary 指定首部字段的请求返回 缓存。即使对相同资源发起请求，但由于 Vary 指定的首部字段不相同， 因此必须要从源服务器重新获取资源。 

# 实体首部字段 

实体首部字段是包含在请求报文和响应报文中的实体部分所使用的 首部，用于补充内容的更新时间等与实体相关的信息。 

## Allow 

首部字段 Allow 用于通知客户端能够支持 Request-URI 指定资源的 所有 HTTP 方法。当服务器接收到不支持的 HTTP 方法时，会以状态码 405 Method Not Allowed 作为响应返回。与此同时，还会把所有能支持 的 HTTP 方法写入首部字段 Allow 后返回。 

## Content-Encoding 

首部字段 Content-Encoding 会告知客户端服务器对实体的主体部分 选用的内容编码方式。内容编码是指在不丢失实体信息的前提下所进行 的压缩。 

## Content-Language 

## Content-Length 

## Content-Location 

## Content-Type 

## Content-Range 

## Last-Modified 

不用说

## Content-MD5 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f685de1a8f07449ba5ff6b46eb6bb5cd~tplv-k3u1fbpfcp-zoom-1.image)

首部字段 Content-MD5 是一串由 MD5 算法生成的值，其目的在于 检查报文主体在传输过程中是否保持完整，以及确认传输到达。 

对报文主体执行 MD5 算法获得的 128 位二进制数，再通过 Base64 编码后将结果写入 Content-MD5 字段值。由于 HTTP 首部无法记录二进 制值，所以要通过 Base64 编码处理。为确保报文的有效性，作为接收 方的客户端会对报文主体再执行一次相同的 MD5 算法。计算出的值与 字段值作比较后，即可判断出报文主体的准确性 

## Expires 

首部字段 Expires 会将资源失效的日期告知客户端。缓存服务器在 接收到含有首部字段 Expires 的响应后，会以缓存来应答请求，在 Expires 字段值指定的时间之前，响应的副本会一直被保存。当超过 指定的时间后，缓存服务器在请求发送过来时，会转向源服务器请求 资源。 

源服务器不希望缓存服务器对资源缓存时，最好在 Expires 字段内 写入与首部字段 Date 相同的时间值。 

但是，当首部字段 Cache-Control 有指定 max-age 指令时，比起首 部字段 Expires，会优先处理 max-age 指令。 

# 为 Cookie 服务的首部字段 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0dbc032dcc23456da8f095d9464a3c58~tplv-k3u1fbpfcp-zoom-1.image)

## Set-Cookie 

```
Set-Cookie: status=enable; expires=Tue, 05 Jul 2011 07:26:31 GMT; path=/; domain=.hackr.jp; 
```

当服务器准备开始管理客户端的状态时，会事先告知各种信息。 下面的表格列举了 Set-Cookie 的字段值 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8289a46782147539d6822773b4298de~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed47dd092a4b4b6c8bacaf4cc41fa314~tplv-k3u1fbpfcp-zoom-1.image)

### expires 属性 

Cookie 的 expires 属性指定浏览器可发送 Cookie 的有效期。 当省略 expires 属性时，其有效期仅限于维持浏览器会话（Session） 时间段内。这通常限于浏览器应用程序被关闭之前。 另外，一旦 Cookie 从服务器端发送至客户端，服务器端就不存在可以显式删除 Cookie 的方法。但可通过覆盖已过期的 Cookie，实现对 客户端 Cookie 的实质性删除操作。 

### path 属性 

Cookie 的 path 属性可用于限制指定 Cookie 的发送范围的文件目 录。不过另有办法可避开这项限制，看来对其作为安全机制的效果不能抱有期待。 

### domain 属性 

通过 Cookie 的 domain 属性指定的域名可做到与结尾匹配一致。比 如，当指定 example.com 后，除 example.com 以外，www.example.com 或 www2.example.com 等都可以发送 Cookie。 因此，除了针对具体指定的多个域名发送 Cookie 之外，不指定 domain 属性显得更安全。 

### secure 属性 

Cookie 的 secure 属性用于限制 Web 页面仅在 HTTPS 安全连接时， 才可以发送 Cookie。 发送 Cookie 时，指定 secure 属性的方法为：

`Set-Cookie: name=value; secure `

以上例子仅当在 https ：//www.example.com/（HTTPS）安全连接的 情况下才会进行 Cookie 的回收。也就是说，即使域名相同，http://www. example.com/（HTTP）也不会发生 Cookie 回收行为。 当省略 secure 属性时，不论 HTTP 还是 HTTPS，都会对 Cookie 进 行回收。 

### HttpOnly 属性 

Cookie 的 HttpOnly 属性是 Cookie 的扩展功能，它使 JavaScript 脚 本无法获得 Cookie。其主要目的为防止跨站脚本攻击（Cross-site scripting，XSS）对 Cookie 的信息窃取。 发送指定 HttpOnly 属性的 Cookie 的方法如下所示。 

`Set-Cookie: name=value; HttpOnly `

通过上述设置，通常从 Web 页面内还可以对 Cookie 进行读取操作。 但使用 JavaScript 的 document.cookie 就无法读取附加 HttpOnly 属性后 的 Cookie 的内容了。因此，也就无法在 XSS 中利用 JavaScript 劫持 Cookie 了。 

虽然是独立的扩展功能，但 Internet Explorer 6 SP1 以上版本等当下 的主流浏览器都已经支持该扩展了。

## Cookie 

首部字段 Cookie 会告知服务器，当客户端想获得 HTTP 状态管理 支持时，就会在请求中包含从服务器接收到的 Cookie。接收到多个 Cookie 时，同样可以以多个 Cookie 形式发送。 

# 其他首部字段 

HTTP 首部字段是可以自行扩展的。所以在 Web 服务器和浏览器的 应用上，会出现各种非标准的首部字段。 接下来，我们就一些最为常用的首部字段进行说明。 

## X-Frame-Options 

首部字段 X-Frame-Options 属于 HTTP 响应首部，用于控制网站内 容在其他 Web 网站的 Frame 标签内的显示问题。其主要目的是为了防 止点击劫持（clickjacking）攻击。 首部字段 X-Frame-Options 有以下两个可指定的字段值。 

-   DENY ：拒绝 
-   SAMEORIGIN ：仅同源域名下的页面（Top-level-browsing-context） 匹配时许可。（比如，当指定 http://hackr.jp/sample.html 页面为 SAMEORIGIN 时，那么 hackr.jp 上所有页面的 frame 都被允许可 加载该页面，而 example.com 等其他域名的页面就不行了） 

## X-XSS-Protection 

首部字段 X-XSS-Protection 属于 HTTP 响应首部，它是针对跨站脚 本攻击（XSS）的一种对策，用于控制浏览器 XSS 防护机制的开关。 首部字段 X-XSS-Protection 可指定的字段值如下。 

● 0 ：将 XSS 过滤设置成无效状态 

● 1 ：将 XSS 过滤设置成有效状态 

## DNT 

首部字段 DNT 属于 HTTP 请求首部，其中 DNT 是 Do Not Track 的简称，意为拒绝个人信息被收集，是表示拒绝被精准广告追踪的一 种方法。 

● 0 ：同意被追踪 

● 1 ：拒绝被追踪 

## P3P 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4d083f07a134a88b1750bae4b7fa689~tplv-k3u1fbpfcp-zoom-1.image)

