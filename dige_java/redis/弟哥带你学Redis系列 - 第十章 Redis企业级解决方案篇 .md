## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 缓存预热

**现象：** 服务器启动后快速宕机

**原因：**

1.  请求数量较高
1.  主从之间数据吞吐量较大，数据同步操作频度较高



**解决：**

前置准备工作:

1．日常例行统计数据访问记录，统计访问频度较高的热点数据

2．利用LRU数据删除策略，构建数据留存队列。例如: storm与kafka配合

准备工作:

3．将统计结果中的数据分类，根据级别，redis优先加载级别较高的热点数据

4．利用分布式多服务器同时进行数据读取，提速数据加载过程

实施:

1．使用脚本程序固定触发数据预热过程

2．如果条件允许，使用了CDN(内容分发网络)，效果会更好

**总结**

缓存预热就是系统启动前，提前将相关的缓存数据直接加载到缓存系统。避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题!用户直接查询事先被预热的缓存数据!

# 缓存雪崩

**现象：**

1．系统平稳运行过程中，忽然数据库连接量激增

2．应用服务器无法及时处理请求

3．大量408，500错误页面出现

4．客户反复刷新页面获取数据

5．数据库崩溃

6．应用服务器崩溃

7．重启应用服务器无效

8. Redis服务器崩溃

9. Redis集群崩溃

10.重启数据库后再次被瞬间流量放倒

**原因：**

1．在一个较短的时间内，缓存中较多的key集中过期

2．此周期内请求访问过期的数据,redis未命中，redis向数据库获取数据

3．数据库同时接收到大量的请求无法及时处理

4. Redis大量请求被积压，开始出现超时现象

5．数据库流量激增，数据库崩溃

6．重启后仍然面对缓存中无数据可用

7. Redis服务器资源被严重占用，Redis服务器崩溃

8. Redis集群呈现崩塌，集群瓦解

9.应用服务器无法及时得到数据响应请求，来自客户端的请求数量越来越多，应用服务器崩溃

10.应用服务器，redis，数据库全部重启，效果不理想

解决：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/908046dac3744699a60845e709afa047~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5bbe0e93eece498385fc463c946818a0~tplv-k3u1fbpfcp-zoom-1.image)

# 缓存击穿


**现象：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/451c2a8897c94cdfa53dcdc77e0ba565~tplv-k3u1fbpfcp-zoom-1.image)

**原因：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d8d28bc433e4b37b5addf27e1d3ab2b~tplv-k3u1fbpfcp-zoom-1.image)



**解决：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebf199d81a144b19ae3096969d28b42c~tplv-k3u1fbpfcp-zoom-1.image)



总结：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3965768439c947a6b68c9c91d648f776~tplv-k3u1fbpfcp-zoom-1.image)

# 缓存穿透

**现象：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6aca7dd3fea24f8e84e3b96bf1e619c8~tplv-k3u1fbpfcp-zoom-1.image)

**原因：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7f648a489fb41bd94f0f7ab7b0a6b54~tplv-k3u1fbpfcp-zoom-1.image)

**分析：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b534995dfd4946eeb8a9409a0a094f2d~tplv-k3u1fbpfcp-zoom-1.image)

**解决：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88a0e1fd3fbc41b9ac68c8058912b2cf~tplv-k3u1fbpfcp-zoom-1.image)

**总结：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a823b429d3c4da7996a0f468284e95b~tplv-k3u1fbpfcp-zoom-1.image)

