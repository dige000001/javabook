## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# redis集群结构设计

## 数据存储设计

当要存储一个key时：

1.  通过算法设计，计算出key应该保存的位置，CRC(key)%16384
1.  将所有的存储空间计划切割成16384份，每台主机保存一部分

-   -   每份代表的是一个存储空间，不是一个key的保存空间

3.  将key按照计算出的结果放到对应的存储空间

当集群增加一台主机时：

1.  将所有的存储空间计划切割成16384份
1.  每个主机将该属于新主机的slot发送给新主机，即数据转交

## 集群内部通信设计

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32a43b80c7c94792891a28069fd79619~tplv-k3u1fbpfcp-zoom-1.image)

# 配置集群

配置文件：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95de8a2afbf444adba5780da11c51f79~tplv-k3u1fbpfcp-zoom-1.image)

开启集群：

需要ruby和rubygems

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a13234b585274e9e8f939959db013a2e~tplv-k3u1fbpfcp-zoom-1.image)

replicas 1表示一个master有一个slave

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a61caec036b04e088a83f5cb184424f3~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/393c8a0e99244caf8e4b088bb12c17de~tplv-k3u1fbpfcp-zoom-1.image)

# 数据操作

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/524ddd9e0d0a42dda9793bcc700c7041~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6fef7304631e4ccf827173468770676b~tplv-k3u1fbpfcp-zoom-1.image)

可以看到，只能使用`redis-cli -c`才能像单机那样使用redis

### cluster节点操作命令

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adcc85b73535433f84ddc79cb53511da~tplv-k3u1fbpfcp-zoom-1.image)



