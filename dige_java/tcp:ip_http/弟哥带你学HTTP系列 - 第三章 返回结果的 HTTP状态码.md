
## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed108892c17b4a1d92a606c2506f4347~tplv-k3u1fbpfcp-zoom-1.image)

# 2XX 成功 

2XX 的响应结果表明请求被正常处理了。 

### 200 ok

表示从客户端发来的请求在服务器端被正常处理了。 

### 204 No Content 

该状态码代表服务器接收的请求已成功处理，但在返回的响应报文 中不含实体的主体部分。另外，也不允许返回任何实体的主体。比如， 当从浏览器发出请求处理后，返回 204 响应，那么浏览器显示的页面不 发生更新。 一般在只需要从客户端往服务器发送信息，而对客户端不需要发送 新信息内容的情况下使用。 

### 206 Partial Content 

该状态码表示客户端进行了范围请求，而服务器成功执行了这部分 的 GET 请求。响应报文中包含由 Content-Range 指定范围的实体内容。 

# 3XX 重定向 

3XX 响应结果表明浏览器需要执行某些特殊的处理以正确处理 请求 

### 301 Moved Permanently 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3003a3190dab4faf877e8d457f1afd21~tplv-k3u1fbpfcp-zoom-1.image)

永久性重定向。该状态码表示请求的资源已被分配了新的 URI，以 后应使用资源现在所指的 URI。也就是说，如果已经把资源对应的 URI 保存为书签了，这时应该按 Location 首部字段提示的 URI 重新保存 

### 302 Found 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66b0faf66e1f45cdabeed6233babcb5e~tplv-k3u1fbpfcp-zoom-1.image)

临时性重定向。该状态码表示请求的资源已被分配了新的 URI，希 望用户（本次）能使用新的 URI 访问。 

和 301 Moved Permanently 状态码相似，但 302 状态码代表的资源 不是被永久移动，只是临时性质的。换句话说，已移动的资源对应的 URI 将来还有可能发生改变。比如，用户把 URI 保存成书签，但不会像 301 状态码出现时那样去更新书签，而是仍旧保留返回 302 状态码的页 面对应的 URI 

### 303 See Other 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca4b8509f9bb4ffcb571f05eb9439fda~tplv-k3u1fbpfcp-zoom-1.image)

该状态码表示由于请求对应的资源存在着另一个 URI，应使用 GET 方法定向获取请求的资源。 303 状态码和 302 Found 状态码有着相同的功能，但 303 状态码明 确表示客户端应当采用 GET 方法获取资源，这点与 302 状态码有区别。 比如，当使用 POST 方法访问 CGI 程序，其执行后的处理结果是希 望客户端能以 GET 方法重定向到另一个 URI 上去时，返回 303 状态码。 虽然 302 Found 状态码也可以实现相同的功能，但这里使用 303 状态码 是最理想的。 

注： 当 301、302、303 响应状态码返回时，几乎所有的浏览器都会把 POST 改成 GET，并删除请求报文内的主体，之后请求会自动再次发送。 301、302 标准是禁止将 POST 方法改变成 GET 方法的，但实际使用 时大家都会这么做。 

### 304 Not Modified 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28fb5f4f66324f0783646cb9e52e4f4b~tplv-k3u1fbpfcp-zoom-1.image)

该状态码表示客户端发送附带条件的请求 A 时，服务器端允许请求访问资源，但未满足条件的情况。304 状态码返回时，不包含任何响应 的主体部分。304 虽然被划分在 3XX 类别中，但是和重定向没有关系。 

注： 附带条件的请求是指采用GET方法的请求报文中包含If-Match，If-ModifiedSince，If-None-Match，If-Range，If-Unmodified-Since中任一首部。 

### 307 Temporary Redirect 

临时重定向。该状态码与 302 Found 有着相同的含义。尽管 302 标准禁止 POST 变换成 GET，但实际使用时大家并不遵守。 307 会遵照浏览器标准，不会从 POST 变成 GET。但是，对于处理响应时的行为，每种浏览器有可能出现不同的情况 

# 4XX 客户端错误 

4XX 的响应结果表明客户端是发生错误的原因所在。 

### 400 Bad Request 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7d33e9f227f4e4486079941dbe1b82c~tplv-k3u1fbpfcp-zoom-1.image)

该状态码表示请求报文中存在语法错误。当错误发生时，需修改请求 的内容后再次发送请求。另外，浏览器会像 200 OK 一样对待该状态码。 

### 401 Unauthorized 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db5bc504dfcf4eb38322e9bfc12ac1c7~tplv-k3u1fbpfcp-zoom-1.image) 该状态码表示发送的请求需要有通过 HTTP 认证（BASIC 认证、 DIGEST 认证）的认证信息。另外若之前已进行过 1 次请求，则表示用户认证失败。 返回含有 401 的响应必须包含一个适用于被请求资源的 WWW-Authenticate 首部用以质询（challenge）用户信息。当浏览器初次接收 到 401 响应，会弹出认证用的对话窗口 

### 403 Forbidden 

该状态码表明对请求资源的访问被服务器拒绝了。服务器端没有必 要给出拒绝的详细理由，但如果想作说明的话，可以在实体的主体部分 对原因进行描述，这样就能让用户看到了。 未获得文件系统的访问授权，访问权限出现某些问题（从未授权的 发送源 IP 地址试图访问）等列举的情况都可能是发生 403 的原因 

### 404 Not Found 

该状态码表明服务器上无法找到请求的资源。除此之外，也可以在 服务器端拒绝请求且不想说明理由时使用。 

# 5XX 服务器错误 

5XX 的响应结果表明服务器本身发生错误。 

### 500 Internal Server Error 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/019976c3cfae4cbfa0b43813dce8ca18~tplv-k3u1fbpfcp-zoom-1.image)

该状态码表明服务器端在执行请求时发生了错误。也有可能是 Web 应用存在的 bug 或某些临时的故障。 

### 503 Service Unavailable 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf686af902704212b499f4cd7474dcf0~tplv-k3u1fbpfcp-zoom-1.image)

该状态码表明服务器暂时处于超负载或正在进行停机维护，现在无 法处理请求。如果事先得知解除以上状况需要的时间，最好写入 RetryAfter 首部字段再返回给客户端。