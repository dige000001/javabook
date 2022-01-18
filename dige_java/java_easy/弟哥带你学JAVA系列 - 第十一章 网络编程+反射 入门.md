## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 基本概念
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84868a92217840f7bf696f6f4e2b68f2~tplv-k3u1fbpfcp-zoom-1.image)

# TCP

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f26350bfcb94b21925daf496b88a8cc~tplv-k3u1fbpfcp-zoom-1.image)

Server端编写

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d843b3df88f3487cb42520b84d587667~tplv-k3u1fbpfcp-zoom-1.image)



Client例子

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/731895275cd6460db20f5f5166696ebc~tplv-k3u1fbpfcp-zoom-1.image)

# UDP

Client例子

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8cf0515f20e0451690f7d8f105c7fbc5~tplv-k3u1fbpfcp-zoom-1.image)

Server例子

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/afb90ccab98c4d6397fa9646105fee84~tplv-k3u1fbpfcp-zoom-1.image)

# 反射

ClassLoader：

<https://www.cnblogs.com/lxk2010012997/p/5221963.html>

java的类是运行时加载的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f317187a9f2f4c3ca61b4fe33a40cae4~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/acb51cb8635d410ab0f13714c482d883~tplv-k3u1fbpfcp-zoom-1.image)

这里应该是Object o = c.newInstance();




Class.forName:<https://blog.csdn.net/syilt/article/details/90706332>

类里的静态语句

<https://blog.csdn.net/wjlwangluo/article/details/76606040>

反射的作用：在运行期获取类的信息（如方法等），并根据该类的Class对象进行调用，在各类框架的使用中，最主要的就是反射

# 工厂模式

<https://zhuanlan.zhihu.com/p/239952896>

<https://zhuanlan.zhihu.com/p/243277598>

本质：对外隐藏构造函数，解耦

