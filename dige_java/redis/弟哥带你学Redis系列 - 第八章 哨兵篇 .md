## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ea3f737f5fb42eaab709ce01b78c230~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fba1bb2b3a9e49849cc4d34a3cf240a9~tplv-k3u1fbpfcp-zoom-1.image)

## 哨兵作用

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5c4fde316dc4973aac50dd4f8839aa9~tplv-k3u1fbpfcp-zoom-1.image)

单数是为了竞选总有胜者

下线的master也会被持续监控，上线后sentinel会让其成为新master的slave

# 启用哨兵模式

## 配置哨兵

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e0e07f1797a40639f9d73122ddbf921~tplv-k3u1fbpfcp-zoom-1.image)




monitor mymaster 127.0.0.1 6379 2

指示 Sentinel 去监视一个名为 mymaster 的主服务器， 这个主服务器的 IP 地址为 127.0.0.1 ， 端口号为 6379 ， 而将这个主服务器判断为失效至少需要 2 个 Sentinel 同意 （只要同意 Sentinel 的数量不达标，自动故障迁移就不会执行）。

down-after-milliseconds mymaster 30000

指定了 Sentinel 认为服务器已经断线所需的毫秒数（判定为主观下线SDOWN）。

parallel-syncs mymaster 1 

指定了在执行故障转移时， 最多可以有多少个从服务器同时对新的主服务器进行同步， 这个数字越小， 完成故障转移所需的时间就越长，但越大就意味着越多的从服务器因为复制而不可用。可以通过将这个值设为 1 来保证每次只有一个从服务器处于不能处理命令请求的状态。

failover-timeout mymaster 180000

选项指定同步超时时间

**主观下线（Subjectively Down， 简称 SDOWN）** 指的是单个 Sentinel 实例对服务器做出的下线判断。\
**客观下线（Objectively Down， 简称 ODOWN）** 指的是多个 Sentinel 实例在对同一个服务器做出 SDOWN 判断， 并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后， 得出的服务器下线判断。

客观下线条件只适用于主服务器： 对于任何其他类型的 Redis 实例， Sentinel 在将它们判断为下线前不需要进行协商， 所以从服务器或者其他 Sentinel 永远不会达到客观下线条件。\
只要一个 Sentinel 发现某个主服务器进入了客观下线状态， 这个 Sentinel 就可能会被其他 Sentinel 推选出， 并对失效的主服务器执行自动故障迁移操作。

# 哨兵工作原理

## 阶段一：监控阶段

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f20e376c79d9476fbee4ca2e61944414~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b39252a27aa1412686586af155573161~tplv-k3u1fbpfcp-zoom-1.image)

在该阶段，sentinel会向master和slave获取信息，同时sentinels之间会构成网络，发布/订阅信息

## 阶段二：通知阶段

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fada0fb6569844e9b526c55723d52b14~tplv-k3u1fbpfcp-zoom-1.image)

其中一个sentinel向master和slave获取信息，然后在sentinel的网络内部发布

## 阶段三：故障转移阶段

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/472ce6a03dcc4d8189652736a8ac6310~tplv-k3u1fbpfcp-zoom-1.image)

**主观下线（Subjectively Down， 简称 SDOWN）** 指的是单个 Sentinel 实例对服务器做出的下线判断。\
**客观下线（Objectively Down， 简称 ODOWN）** 指的是多个 Sentinel 实例在对同一个服务器做出 SDOWN 判断， 并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后， 得出的服务器下线判断。

客观下线条件只适用于主服务器： 对于任何其他类型的 Redis 实例， Sentinel 在将它们判断为下线前不需要进行协商， 所以从服务器或者其他 Sentinel 永远不会达到客观下线条件。\
只要一个 Sentinel 发现某个主服务器进入了客观下线状态， 这个 Sentinel 就可能会被其他 Sentinel 推选出， 并对失效的主服务器执行自动故障迁移操作。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/125cc010b2124e13a552cafc28b3cb0d~tplv-k3u1fbpfcp-zoom-1.image)

master挂了之后，sentinel之间会进行竞选，选出的sentinel要去选出新的master，流程如下：

1.  从服务器中挑选master

-   在线的
-   响应快的

<!---->

-   与原master断开时间短的
-   优先原则

<!---->

-   -   优先级
    -   offset

<!---->

-   -   runid

2.  发送指令( sentinel )

-   向新的master发送slaveof no one
-   向其他slave发送slaveof新master lP端口

