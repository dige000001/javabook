## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 简介

主从复制即将master中的数据即时、有效的复制到slave中

特征：一个master可以拥有多个slave，一个slave只对应一个master



**master:**

-   写数据
-   执行写操作时，将出现变化的数据自动同步到slave

<!---->

-   读数据(可忽略)

**slave:**

-   读数据
-   写数据(禁止)

## 作用

-   **读写分离:** master写、slave读，提高服务器的读写负载能力
-   **负载均衡:** 基于主从结构，配合读写分离，由slave分担master负载，并根据需求的变化，改变slave的数量，

通过多个从节点分担数据读取负载，大大提高Redis服务器并发量与数据吞吐量

-   **故障恢复:** 当master出现问题时，由slave提供服务，实现快速的故障恢复
-   **数据冗余: ** 实现数据热备份，是持久化之外的一种数据冗余方式

<!---->

-   **高可用基石:** 基于主从复制，构建哨兵模式与集群，实现Redis的高可用方案

# 主从复制工作流程




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06e2201ea03946c98d138d2ee4ed8f4a~tplv-k3u1fbpfcp-zoom-1.image)

## 阶段一：建立连接

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f903193b7d24950971f5e7a4d5778fa~tplv-k3u1fbpfcp-zoom-1.image)

### 连接指令

方式一:客户端发送命令

slaveof <masterip> <masterport>

方式二︰启动服务器参数

redis-server --slaveof <masterip> <masterport>

方式三:服务器配置

slaveof <masterip> <masterport>

### 授权访问

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/86653155a5c44abbaea4e2aed4df1dac~tplv-k3u1fbpfcp-zoom-1.image)

## 阶段二：数据同步

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c92488bb89e473ab84d8af3c43a9f39~tplv-k3u1fbpfcp-zoom-1.image)

### 注意事项

#### master注意事项

1.  如果master数据量巨大，数据同步阶段应避开流量高峰期，避免造成master阻塞，影响业务正常执
1.  复制缓冲区大小设定不合理，会导致数据溢出。如进行全量复制周期太长，进行部分复制时发现数据已\
    经存在丢失的情况，必须进行第二次全量复制，可能致使slave陷入死循环状态。

repl-backlog-size 1mb

3.  master单机内存占用主机内存的比例不应过大，建议使用50%-70%的内存，留下30%-50%的内存用于执\
    行bgsave命令和创建复制缓冲区

#### slave注意事项

1.  为避免slave进行全量复制、部分复制时服务器响应阻塞或数据不同步，建议关闭此期间的外服务

slave-serve-stale-data yes | no

2.  数据同步阶段，master发送给slave信息可以理解master是slave的一个客户端，主动向slave发送命令

<!---->

3.  多个slave同时对master请求数据同步，master发送的RDB文件增多，会对带宽造成巨大冲击，如果master带宽不足，因此数据同步需要根据业务需求，适量错峰
3.  slave过多时，建议调整拓扑结构，由一主多从结构变为树状结构，中间的节点既是master，也是slave。注意使用树状结构时，由于层级深度，导致深度越高的slave与最顶层master间数据同步延迟较大，数据一致性变差，应谨慎选择

## 阶段三：命令传播阶段

-   当master数据库状态被修改后，导致主从服务器数据库状态不一致，此时需要让主从数据同步到一致的状态，同步的动作称为命令传播
-   master将接收到的数据变更命令发送给slave，slave接收命令后执行命令

### 命令传播阶段的部分复制

-   部分复制的三个核心要素

<!---->

-   -   服务器的运行id (run id)
    -   主服务器的复制积压缓冲区

<!---->

-   -   主从服务器的复制偏移量offset



-   复制策略：

<!---->

-   -   如果offset偏移量之后的数据（也即是偏移量offset+1开始的数据）仍然存在于复制积压缓冲区里面，那么主服务器将对从服务器执行部分重同步操作。
    -   相反，如果offset偏移量之后的数据已经不存在于复制积压缓冲区，那么主服务器将对从服务器执行完整重同步操作。

#### 运行id

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9d77278d15142f581c0da0d2854181e~tplv-k3u1fbpfcp-zoom-1.image)

#### 复制缓冲区

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e7cbd415dba4639ba420b12a7ffcc37~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/849a5a9d8ba9432d8e65d5d489557f88~tplv-k3u1fbpfcp-zoom-1.image)

#### 偏移量

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc6592e779b54ab7bad6ce0bf18d3ed5~tplv-k3u1fbpfcp-zoom-1.image)

### 数据同步+命令传播阶段工作流程



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5787d688c8343188401f24c1578b2c0~tplv-k3u1fbpfcp-zoom-1.image)



### 心跳机制

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c8d0cc1affd40f6b601a9e553b18297~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f6babdba6f14388bdf490c9359ced4d~tplv-k3u1fbpfcp-zoom-1.image)

### 注意事项

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c0a83ba81874ba2aa43ccad7fd86ad4~tplv-k3u1fbpfcp-zoom-1.image)

# 主从复制常见问题

## 频繁的全量复制

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0854e5ed4504208bb556bf9d9a92f1f~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a999f7e97724d8283c7c97cf9fa773f~tplv-k3u1fbpfcp-zoom-1.image)

## 频繁的网络中断

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b82071f1661c42c3ac87996861c5e03c~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3922bdb456ff46dfacc0304abc8b5bac~tplv-k3u1fbpfcp-zoom-1.image)

## 数据不一致

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfc730875986428cb32ebd3b42d15b92~tplv-k3u1fbpfcp-zoom-1.image)

