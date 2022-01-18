## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# HTTP 报文 

用于 HTTP 协议交互的信息被称为 HTTP 报文。请求端（客户端）的 HTTP 报文叫做请求报文，响应端（服务器端）的叫做响应报文。HTTP 报文本身是由多行（用 CR+LF 作换行符）数据构成的字符串文本。 HTTP 报文大致可分为报文首部和报文主体两块。两者由最初出现 的空行（CR+LF）来划分。通常，并不一定要有报文主体。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4255d5aed1646fa81829f87b472f2b0~tplv-k3u1fbpfcp-zoom-1.image)

# 请求报文及响应报文的结构 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03181caa1f244ca9a5e083ca7c9e1a1a~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c8e54fa7e1946bba85e7d6e8439d451~tplv-k3u1fbpfcp-zoom-1.image)

# 分割发送的分块传输编码 

在 HTTP 通信过程中，请求的编码实体资源尚未全部传输完成之 前，浏览器无法显示请求页面。在传输大容量数据时，通过把数据分割 成多块，能够让浏览器逐步显示页面。 这种把实体主体分块的功能称为分块传输编码（Chunked Transfer Coding）。 

# 发送多种数据的多部分对象集合 

发送邮件时，我们可以在邮件里写入文字并添加多份附件。这是因为采用了 MIME（Multipurpose Internet Mail Extensions，多用途因特网 邮件扩展）机制，它允许邮件处理文本、图片、视频等多个不同类型的 数据。例如，图片等二进制数据以 ASCII 码字符串编码的方式指明，就 是利用 MIME 来描述标记数据类型。而在 MIME 扩展中会使用一种称 为多部分对象集合（Multipart）的方法，来容纳多份不同类型的数据。 

相应地，HTTP 协议中也采纳了多部分对象集合，发送的一份报文主体内可含有多类型实体。通常是在图片或文本文件等上传时使用。 

# 获取部分内容的范围请求 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ae4e9084032486383b7706bd87a0655~tplv-k3u1fbpfcp-zoom-1.image)

# 内容协商返回最合适的内容 

同一个 Web 网站有可能存在着多份相同内容的页面。比如英语版 和中文版的 Web 页面，它们内容上虽相同，但使用的语言却不同。 当浏览器的默认语言为英语或中文，访问相同 URI 的 Web 页面时， 则会显示对应的英语版或中文版的 Web 页面。这样的机制称为内容协商（Content Negotiation）。 

内容协商机制是指客户端和服务器端就响应的资源内容进行交涉， 然后提供给客户端最为适合的资源。内容协商会以响应资源的语言、字 符集、编码方式等作为判断的基准。 包含在请求报文中的某些首部字段（如下）就是判断的基准。这些首部字段的详细说明请参考下一章 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b19704d9ad524ae3b04f942536d4b5c9~tplv-k3u1fbpfcp-zoom-1.image)



内容协商技术有以下 3 种类型。 

-   **服务器驱动协商**（Server-driven Negotiation） 由服务器端进行内容协商。以请求的首部字段为参考，在服务器端自动处理。但对用户来说，以浏览器发送的信息作为判定的依据， 并不一定能筛选出最优内容。 
-   **客户端驱动协商**（Agent-driven Negotiation） 由客户端进行内容协商的方式。用户从浏览器显示的可选项列表中 手动选择。还可以利用 JavaScript 脚本在 Web 页面上自动进行上述 选择。比如按 OS 的类型或浏览器类型，自行切换成 PC 版页面或 手机版页面。 

<!---->

-   **透明协商**（Transparent Negotiation） 是服务器驱动和客户端驱动的结合体，是由服务器端和客户端各自进行内容协商的一种方法。

