## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

Redis集群是Redis提供的分布式数据库方案，集群通过分片( sharding)来进行数据共享，并提供复制和故障转移功能。

本节将对集群的节点、槽指派、命令执行、重新分片、转向、故障转移、消息等各个方面进行介绍。

# 节点

# 启动节点

一个节点就是一个运行在集群模式下的Redis服务器，Redis服务器在启动时会根据cluster-enabled配置选项是否为yes来决定是否开启服务器的集群模式，如图17-6所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9451687083be4c808cfb131bfd8d7af1~tplv-k3u1fbpfcp-zoom-1.image)

节点（运行在集群模式下的Redis服务器）会继续使用所有在单机模式中使用的服务器组件，比如说:

-   -   节点会继续使用文件事件处理器来处理命令请求和返回命令回复。
    -   节点会继续使用时间事件处理器来执行servercron函数，而serverCron函数又会调用集群模式特有的clustercron 函数。clustercron函数负责执行在集群模式下需要执行的常规操作，例如向集群中的其他节点发送Gossip消息，检查节点是否断线，或者检查是否需要对下线节点进行自动故障转移等。

<!---->

-   -   节点会继续使用数据库来保存键值对数据，键值对依然会是各种不同类型的对象。
    -   节点会继续使用RDB持久化模块和AOF持久化模块来执行持久化工作。

<!---->

-   -   节点会继续使用发布与订阅模块来执行PUBLISH、SUBSCRIBE等命令。
    -   节点会继续使用复制模块来进行节点的复制工作。

<!---->

-   -   节点会继续使用Lua脚本环境来执行客户端输入的Lua脚本。

除此之外，节点会继续使用redisserver结构来保存服务器的状态，使用redisclient结构来保存客户端的状态，至于那些只有在集群模式下才会用到的数据，节点将它们保存到了cluster.h/clusterNode结构、cluster.h/clusterLink结构，以及cluster.h/clusterstate结构里面，接下来的一节将对这三种数据结构进行介绍。

## 集群数据结构

`clusterNode`结构保存了一个节点的当前状态，比如节点的创建时间、节点的名字、节点当前的配置纪元、节点的IP地址和端口号等等。

每个节点都会使用一个clusterNode结构来记录自己的状态，并**为集群中的所有其他节点（包括主节点和从节点）都创建一个相应的clusterNode结构，** 以此来记录其他节点的状态，

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2dfb16d7e112496c83bb680216132185~tplv-k3u1fbpfcp-zoom-1.image)

clusterNode结构的link属性是一个clusterLink结构，该结构保存了连接节点所需的有关信息，比如套接字描述符，输人缓冲区和输出缓冲区:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d45b49a730c940dbbfa8bc44b416030d~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b47690f4bc21412f819f7803495f7c50~tplv-k3u1fbpfcp-zoom-1.image)

最后，每个节点都保存着一个clusterstate结构，这个结构记录了在当前节点的视角下，集群目前所处的状态，例如集群是在线还是下线，集群包含多少个节点，集群当前的配置纪元，诸如此类:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/829e862403f54d039b4dead40cb82920~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4522b69db6bd431e84db940bb5f8d819~tplv-k3u1fbpfcp-zoom-1.image)

## cluster meet命令的实现

通过向节点A发送`CLUSTER MEET IP PORT`命令，客户端可以让接收命令的节点A将另一个节点B添加到节点A 当前所在的集群里面

节点A将与节点B进行握手( handshake )，以此来确认彼此的存在，并为将来的进一步通信打好基础:

1.  节点A会为节点B创建一个clusterNode结构，并将该结构添加到自己的clusterState.nodes字典里面。
1.  之后，节点A将根据CLUSTER MEET命令给定的P地址和端口号，向节点B发送一条MEET消息（ message )。

<!---->

3.  如果一切顺利，节点B将接收到节点A发送的MEET消息，节点B会为节点A创建一个clusterNode结构，并将该结构添加到自己的clusterstate.nodes字典里面。
3.  之后，节点B将向节点A返回一条PONG消息。

<!---->

5.  如果一切顺利，节点A将接收到节点B返回的 PONG消息，通过这条PONG消息节点A可以知道节点B已经成功地接收到了自己发送的MEET消息。
5.  之后，节点A将向节点B返回一条PING消息。

<!---->

7.  如果一切顺利，节点B将接收到节点A返回的PING消息，通过这条PING消息节点B可以知道节点A已经成功地接收到了自己返回的 PONG消息，握手完成。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24273b45e77345458438b9cb94686f87~tplv-k3u1fbpfcp-zoom-1.image)

之后，节点A会将节点B的信息通过Gossip协议传播给集群中的其他节点，让其他节点也与节点B进行握手，最终，经过一段时间之后，节点B会被集群中的所有节点认识。

# 槽指派

Redis集群通过分片的方式来保存数据库中的键值对:集群的整个数据库被分为16384个槽( slot )，数据库中的每个键都属于这16384个槽的其中一个，集群中的每个节点可以处理0个或最多16384个槽。

通过向节点发送`CLUSTER ADDSLOTS`命令，我们可以将一个或多个槽指派（ assign )给节点负责:

CLUSTER ADDSLOTS <slot> [slot ...]

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ada7693c4c2e4b9bb54729eb820274fb~tplv-k3u1fbpfcp-zoom-1.image)

当数据库中的16384个槽都有节点在处理时，集群处于上线状态（ ok )；相反地，如果数据库中有任何一个槽没有得到处理，那么集群处于下线状态（ fail )。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb34a88355df40ba99d81dd62f997975~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08c7d324e82246e5ba904e75633b239a~tplv-k3u1fbpfcp-zoom-1.image)

本节接下来的内容将首先介绍节点保存槽指派信息的方法，以及节点之间传播槽指派信息的方法，之后再介绍CLUSTER ADDSLOTS命令的实现。

## 记录节点的槽指派信息

clusterNode 结构的slots属性和 numslot属性记录了节点负责处理哪些槽:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58619723028f49249a47313d8ba883d5~tplv-k3u1fbpfcp-zoom-1.image)

slots属性是一个二进制位数组（bit array），这个数组的长度为16384/8=2048个字节，共包含16384个二进制位。

Redis 以0为起始索引，16383为终止索引，对slots数组中的16384个二进制位进行编号，并根据索引i上的二进制位的值来判断节点是否负责处理槽i:

如果slots数组在索引i上的二进制位的值为1，那么表示节点负责处理槽i。如果slots 数组在索引i上的二进制位的值为0，那么表示节点不负责处理槽i。

图17-9展示了一个slots数组示例：这个数组索引0至索引7上的二进制位的值都为1，其余所有二进制位的值都为0，这表示节点负责处理槽0至槽7。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13ca0f8397534c9cbf47f9fb70c17a93~tplv-k3u1fbpfcp-zoom-1.image)

## 传播节点的槽指派信息

一个节点除了会将自己负责处理的槽记录在clusterNode结构的slots属性和numslots属性之外，它还会将自己的slots数组通过消息发送给集群中的其他节点，以此来告知其他节点自己目前负责处理哪些槽。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19047a76552c4dcba84429f2104ef241~tplv-k3u1fbpfcp-zoom-1.image)

当节点A通过消息从节点B那里接收到节点B的slots数组时，节点A会在自己的clusterstate.nodes字典中查找节点B对应的clusterNode结构，并对结构中的slots数组进行保存或者更新。

因为集群中的每个节点都会将自己的slots数组通过消息发送给集群中的其他节点，并且每个接收到slots数组的节点都会将数组保存到相应节点的clusterNode结构里面，因此，集群中的每个节点都会知道数据库中的16384个槽分别被指派给了集群中的哪些节点。

## 记录集群所有槽的指派信息

clusterstate结构中的slots数组记录了集群中所有16384个槽的指派信息:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1870a067a4f14e31b3d95e28846f5f70~tplv-k3u1fbpfcp-zoom-1.image)

slots数组包含16384个项，每个数组项都是一个指向clusterNode结构的指针:

-   -   如果slots[i]指针指向NULL，那么表示槽i尚未指派给任何节点。
    -   如果slots [i〕指针指向一个clusterNode 结构，那么表示槽i已经指派给了clusterNode结构所代表的节点。

**这么做的好处是：如果需要检查某个slot属于谁，则只需要O(1)的复杂度，而不用遍历所有clusterNode。**

**另一方面，如果Redis不使用clusterNode.slots数组，而单独使用clusterstate.slots数组的话，那么每次要将节点A的槽指派信息传播给其他节点时，程序必须先遍历整个clusterstate.slots数组，记录节点A负责处理哪些槽，然后才能发送节点A的槽指派信息，这比直接发送clusterNode.slots数组要麻烦和低效得多。**

## CLUSTER ADDSLOTS命令的实现

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c856a15ff24c44e287fee71c839f5a8a~tplv-k3u1fbpfcp-zoom-1.image)

# 在集群中执行命令

在对数据库中的16384个槽都进行了指派之后，集群就会进入上线状态，这时客户端就可以向集群中的节点发送数据命令了。

当客户端向节点发送与数据库键有关的命令时，接收命令的节点会计算出命令要处理的数据库键属于哪个槽,并检查这个槽是否指派给了自己:

-   -   如果键所在的槽正好就指派给了当前节点，那么节点直接执行这个命令。
    -   如果键所在的槽并没有指派给当前节点，那么节点会向客户端返回一个MOVED错误，指引客户端转向（ redirect）至正确的节点，并再次发送之前想要执行的命令。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e116c4d916a41c3a941568f4f1ec2dd~tplv-k3u1fbpfcp-zoom-1.image)

## 计算属于哪个槽

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a09c387eec8492d8826489e91766bb3~tplv-k3u1fbpfcp-zoom-1.image)

其中 CRC16(key)语句用于计算键key的CRC-16校验和，而& 16383语句则用于计算出一个介于0至16383之间的整数作为键key的槽号。

## 判断槽是否由当前节点负责处理

-   如果clusterstate.slots[ i]等于clusterstate.myself，那么说明槽i由当前节点负责，节点可以执行客户端发送的命令。
-   如果clusterstate.slots [ i]不等于clusterstate.myself，那么说明槽i并非由当前节点负责，节点会根据clusterState.slots[ i]指向的clusterNode结构所记录的节点IP和端口号，向客户端返回MOVED错误，指引客户端转向至正在处理槽i的节点。

## MOVED错误

当节点发现键所在的槽并非由自己负责处理的时候，节点就会向客户端返回一个MOVED错误，指引客户端转向至正在负责槽的节点。

MOVED错误的格式为:

MOVED <slot> <ip> :<port>

其中slot为键所在的槽，而ip和port则是负责处理槽slot的节点的IP地址和端口号。例如错误:

MOVED 10086 127.0.0.1: 7002

表示槽10086正由IP地址为127.0.0.1，端口号为7002的节点负责。

当客户端接收到节点返回的MOVED错误时，客户端会根据MOVED错误中提供的IP地址和端口号，转向至负责处理槽slot的节点，并向该节点重新发送之前想要执行的命令。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6068e5117ad8450fb1bffeffc24d8ad5~tplv-k3u1fbpfcp-zoom-1.image)

一个集群客户端通常会与集群中的多个节点创建套接字连接，而所谓的节点转向实际上就是换一个套接字来发送命令。一个集群客户端通常会与集群中的多个节点创建套接字连接，而所谓的节点转向实际上就是换一个套接字来发送命令.

如果客户端尚未与想要转向的节点创建套接字连接，那么客户端会先根据MOVED错误提供的IP地址和端口号来连接节点,然后再进行转向。如果客户端尚未与想要转向的节点创建套接字连接，那么客户端会先根据移动了错误提供的IP地址和端口号来连接节点，然后再进行转向。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a98fccfe836043308bea704e0c918540~tplv-k3u1fbpfcp-zoom-1.image)

## 节点数据库的实现

集群节点保存键值对以及键值对过期时间的方式，与第9章里面介绍的单机Redis服务器保存键值对以及键值对过期时间的方式完全相同。

节点和单机服务器在数据库方面的一个区别是，**节点只能使用0号数据库**，而单机Redis服务器则没有这一限制。

另外，除了将键值对保存在数据库里面之外，节点还会用clusterstate结构中的slots_to_keys跳跃表来保存槽和键之间的关系:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6bf2031b7834b56bc061d0e0fbbcf67~tplv-k3u1fbpfcp-zoom-1.image)

slots_to_keys跳跃表每个节点的分值( score）都是一个槽号，而每个节点的成员( member)都是一个数据库键：

-   每当节点往数据库中添加一个新的键值对时，节点就会将这个键以及键的槽号关联到slots_to_keys跳跃表。
-   当节点删除数据库中的某个键值对时，节点就会在slots_to_keys跳跃表解除被删除键与槽号的关联。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b303bad4595f46e885f27e78469e0909~tplv-k3u1fbpfcp-zoom-1.image)

注意，一个slot可能对应多个键，而不是一个键

# 重新分片

Redis集群的重新分片操作可以将任意数量已经指派给某个节点（源节点）的槽改为指派给另一个节点（目标节点)，并且相关槽所属的键值对也会从源节点被移动到目标节点。

重新分片操作可以在线( online)进行，在重新分片的过程中，集群不需要下线，并且源节点和目标节点都可以继续处理命令请求。

## 重新分片原理

Redis集群的重新分片操作是由Redis的集群管理软件redis-trib负责执行的，Redis提供了进行重新分片所需的所有命令，而redis-trib 则通过向源节点和目标节点发送命令来进行重新分片操作。

redis-trib对集群的**单个槽slot**进行重新分片的步骤如下:

1.  redis-trib对目标节点发送`CLUSTER SETSLOT <slot> IMPORTING <source_id>`命令，让目标节点准备好从源节点导入(import)属于槽slot的键值对。
1.  redis-trib对源节点发送`CLUSTER SETSLOT <slot> MIGRATING <target_id>`命令，让源节点准备好将属于槽slot的键值对迁移(migrate )至目标节点。

<!---->

3.  redis-trib向源节点发送`CLUSTER GETKEYSINSLOT <slot> <count>`命令，获得最多count个属于槽slot的键值对的键名( key name )。
3.  对于步骤3获得的每个键名，redis-trib都向源节点发送一个

`MIGRATE<target_ip> <target_port> <key_name> 0 <timeout>`命令，将被选中的键原子地从源节点迁移至目标节点。

5.  重复执行步骤3和步骤4，直到源节点保存的所有属于槽slot的键值对都被迁移至目标节点为止。每次迁移键的过程如图17-24所示。
5.  redis-trib向集群中的任意一个节点发送`CLUSTER SETSLOT<slot> NODE<target_id>`命令，将槽slot指派给目标节点，这一指派信息会通过消息发送至整个集群，最终集群中的所有节点都会知道槽slot已经指派给了目标节点。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33ab6ba10a724680a1484b5597edb8e5~tplv-k3u1fbpfcp-zoom-1.image)

如果重新分片涉及多个槽，那么redis-trib将对每个给定的槽分别执行上面给出的步骤。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e03bb56bcc45428dac514dbc4cc9fa0d~tplv-k3u1fbpfcp-zoom-1.image)

##

## CLUSTER SETSLOT IMPORTING命令的实现

clusterState结构的importing_slots_from数组记录了当前节点正在从其他节点导入的槽:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae39ac3ee08d48aea9d3a842c080088e~tplv-k3u1fbpfcp-zoom-1.image)

如果importing_slots_from[i]的值不为NULL，而是指向一个clusterNode结构，那么表示当前节点正在从clusterNode所代表的节点导人槽i。

在对集群进行重新分片的时候，向目标节点发送命令`CLUSTER SETSLOT <i> IMPORTING <source_id>`

可以将目标节点clusterstate.importing_slots_from[i]的值设置为source_id所代表节点的clusterNode结构。

## CLUSTER SETSLOT MIGRATING命令的实现

clusterstate结构的migrating_slots_to数组记录了当前节点正在迁移至其他节点的槽:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52effa66660140b4a2a1676a3db0a379~tplv-k3u1fbpfcp-zoom-1.image)

如果migrating_slots_to[i]的值不为NUIL，而是指向一个clusterNode结构,那么表示当前节点正在将槽i迁移至clusterNode所代表的节点。

在对集群进行重新分片的时候,向源节点发送命令:`CLUSTER SETSLOT <i> MIGRATING <target_id>`

可以将源节点clusterstate.migrating_slots_to[i]的值设置为target_id所代表节点的clusterNode结构。

## ASK错误

在进行重新分片期间，源节点向目标节点迁移一个槽的过程中，可能会出现这样一种情况：属于被迁移槽的一部分键值对保存在源节点里面，而另一部分键值对则保存在目标节点里面。

-   如果节点收到一个关于键key的命令请求，并且键key所属的槽i正好就指派给了这个节点，那么节点会尝试在自己的数据库里查找键key，如果找到了的话，节点就直接执行客户端发送的命令。
-   与此相反，如果节点没有在自己的数据库里找到键key，那么节点会检查自己的`clusterstate.migrating_slots_to[i]`，看键key所属的槽i是否正在进行迁移，如果槽i的确在进行迁移的话，那么节点会向客户端发送一个ASK错误，引导客户端到正在导入槽i的节点去查找键key。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/948aea1180bb4d40b34f6e008755f383~tplv-k3u1fbpfcp-zoom-1.image)

## ASKING命令

ASKING命令唯一要做的就是打开发送该命令的客户端的`REDIS_ASKING`标识

在一般情况下，如果客户端向节点发送一个关于槽i的命令，而槽i又没有指派给这个节点的话，那么节点将向客户端返回一个MOVED错误；但是，**如果节点的clusterstate.importing_slots_from[i]显示节点正在导入槽i，并且发送命令的客户端带有REDIS_ASKING标识，那么节点将破例执行这个关于槽i的命令一次**，要注意的是，客户端的REDIS_ASKING标识是一个一次性标识，当节点执行了一个带有REDIS ASKING标识的客户端发送的命令之后，客户端的REDIS_ASKING标识就会被移除。

图17-31展示了这个判断过程。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/418ab9a457874c5da8e6a16634614ea2~tplv-k3u1fbpfcp-zoom-1.image)

当客户端接收到ASK错误并转向至正在导入槽的节点时，客户端会先向节点发送一个ASKING命令，然后才重新发送想要执行的命令，这是因为如果客户端不发送ASKING命令，而直接发送想要执行的命令的话，那么客户端发送的命令将被节点拒绝执行，并返回MOVED错误。

总结：ASK错误是源节点发给客户端的，让客户端去目标节点找key，客户端想要从目标节点那获得这个正在迁移的key时，需要先发送ASKING命令，让目标节点愿意执行GET请求，才能发送GET命令

# 复制与故障转移

Redis集群中的节点分为主节点( master )和从节点( slave )，其中主节点用于处理槽，而从节点则用于复制某个主节点，并在被复制的主节点下线时，代替下线主节点继续处理命令请求。**主节点可以不止一个**

****

举个例子，对于包含7000、7001、7002、7003四个主节点的集群来说，我们可以将7004、7005两个节点添加到集群里面，并将这两个节点设定为节点7000的从节点，如图17-32所示（图中以双圆形表示主节点，单圆形表示从节点)。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/941534ecefc148baa261dce76ec6717d~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd48b517698445c7a4d2f4c23da40138~tplv-k3u1fbpfcp-zoom-1.image)

如果这时，节点7000进入下线状态，那么集群中仍在正常运作的几个主节点将在节点7000的两个从节点——节点7004和节点7005中选出一个节点作为新的主节点，这个新的主节点将接管原来节点7000负责处理的槽，并继续处理客户端发送的命令请求。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/858fc2092e9d4235ae6064657adb9132~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e010343879674d3a858d772c410ca0b3~tplv-k3u1fbpfcp-zoom-1.image)

如果在故障转移完成之后，下线的节点7000重新上线，那么它将成为节点7004的从节点

## 设置从节点

向一个节点发送命令:`CLUSTER REPLICATE <node_id>`可以让接收命令的节点成为node_id所指定节点的从节点，并开始对主节点进行复制:

-   -   接收到该命令的节点首先会在自己的clusterstate.nodes字典中找到node_id所对应节点的clusterNode结构，并将自己的clusterstate.myself.slaveof指针指向这个结构，以此来记录这个节点正在复制的主节点:![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4dce1f11a06c44699d2d670f942e8b62~tplv-k3u1fbpfcp-zoom-1.image)
    -   然后节点会修改自己在clusterstate.myself.flags 中的属性，关闭原本的REDIS_NODE_MASTER标识，打开REDIS_NODE_SLAVE标识，表示这个节点已经由原来的主节点变成了从节点。

<!---->

-   -   最后，节点会调用复制代码，并根据clusterstate.myself.slaveof指向的clusterNode结构所保存的IP地址和端口号，对主节点进行复制。因为节点的复制功能和单机Redis服务器的复制功能使用了相同的代码，所以让从节点复制主节点相当于向从节点发送命令`SLAVEOF <master_ip> <master _port>。`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2aaaa0d6acd2447fb9bfa1d6c252ca1d~tplv-k3u1fbpfcp-zoom-1.image)

一个节点成为从节点，并开始复制某个主节点这一信息会通过消息发送给集群中的其他节点，最终集群中的所有节点都会知道某个从节点正在复制某个主节点。

集群中的所有节点都会在代表主节点的clusterNode结构的slaves属性和numslaves属性中记录正在复制这个主节点的从节点名单:

```
struct clusterNode {
	// ...
    
	//正在复制这个主节点的从节点数量
    int numslaves;
    
	//一个数组
	//每个数组项指向一个正在复制这个主节点的从节点的clusterNode结构
    struct clusterNode **slaves;
    
	//……
};
```

**这个clusterNode是每个节点的clusterState里nodes字典里保存的数据结构**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6c3ea241fe84419b661ec563fd32376~tplv-k3u1fbpfcp-zoom-1.image)

## 故障检测

集群中的每个节点都会定期地向集群中的其他节点发送PING消息，以此来检测对方是否在线，如果接收PING消息的节点没有在规定的时间内，向发送PING消息的节点返回PONG消息，那么发送PING消息的节点就会将接收PING消息的节点标记为疑似下线( probable fail，PFAIL)。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0697813871141a28a8ee17bb5e295f6~tplv-k3u1fbpfcp-zoom-1.image)

集群中的各个节点会通过互相发送消息的方式来交换集群中各个节点的状态信息，例如某个节点是处于在线状态、疑似下线状态（PFAIL)，还是已下线状态（ FAIL )。

当一个主节点A通过消息得知主节点B认为主节点C进入了疑似下线状态时，主节点A会在自己的clusterstate.nodes字典中找到主节点C所对应的clusterNode结构，并将主节点B的下线报告( failure report）添加到clusterNode结构的fail_reports链表里面:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de325cb565884a47a1f6d8ad7e808212~tplv-k3u1fbpfcp-zoom-1.image)

每个下线报告由一个clusterNodeFailReport结构表示:

```
struct clusterNodeFailReport {
    
	//报告目标节点已经下线的节点，注意是报告节点，而不是下线节点
    struct clusterNode *node;
    
	//最后一次从node节点收到下线报告的时间
    //程序使用这个时间戳来检查下线报告是否过期
    //(与当前时间相差太久的下线报告会被删除)
    mstime_t time;
    
} typedef clusterNodeFailReport;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6300ee9ff10d4fc7b9c2e70faa2e55c0~tplv-k3u1fbpfcp-zoom-1.image)

如果在一个集群里面，半数以上负责处理槽的主节点都将某个主节点x报告为疑似下线，那么这个主节点x将被标记为已下线(FAIL)，将主节点x标记为已下线的节点会向集群广播一条关于主节点x的FAIL消息，所有收到这条FAIL消息的节点都会立即将主节点x标记为已下线。

举个例子，对于图17-38所示的下线报告来说，主节点7002和主节点7003都认为主节点7000进入了下线状态，并且主节点7001也认为主节点7000进入了疑似下线状态（代表主节点7000的结构打开了REDIS_NODE_PFAIL标识)，综合起来，在集群四个负责处理槽的主节点里面，有三个都将主节点7000标记为下线，数量已经超过了半数，所以主节点7001会将主节点7000标记为已下线，并向集群广播一条关于主节点7000的FAIL消息，如图17-39所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f480329a5a72485bad217bc09a1ef126~tplv-k3u1fbpfcp-zoom-1.image)

## 故障转移

当一个从节点发现自己正在复制的主节点进入了已下线状态时，从节点将开始对下线主节点进行故障转移，以下是故障转移的执行步骤:

1.  复制下线主节点的所有从节点里面，会有一个从节点被选中。
1.  被选中的从节点会执行`SLAVEOF no one`命令，成为新的主节点。

<!---->

3.  新的主节点会撤销所有对已下线主节点的槽指派，并将这些槽全部指派给自己。
3.  新的主节点向集群广播一条PONG消息，这条PONG消息可以让集群中的其他节点立即知道这个节点已经由从节点变成了主节点，并且这个主节点已经接管了原本由已下线节点负责处理的槽。

<!---->

5.  新的主节点开始接收和自己负责处理的槽有关的命令请求，故障转移完成。

## 选举新的主节点

新的主节点是通过选举产生的。

以下是集群选举新的主节点的方法:

1.  集群的配置纪元是一个自增计数器，它的初始值为0。
1.  当集群里的某个节点开始一次故障转移操作时，集群配置纪元的值会被增一。

<!---->

3.  对于每个配置纪元，集群里每个负责处理槽的主节点都有一次投票的机会，而第一个向主节点要求投票的从节点将获得主节点的投票。
3.  当从节点发现自己正在复制的主节点进入已下线状态时，从节点会向集群广播一条`CLUSTERMSG_TYPE_FAILOVER_AUTH_REQUEST`消息，要求所有收到这条消息、并且具有投票权的主节点向这个从节点投票。

<!---->

5.  如果一个主节点具有投票权（它正在负责处理槽)，并且这个主节点尚未投票给其他从节点，那么主节点将向要求投票的从节点返回一条`CLUSTERMSG_TYPE_FAILOVERAUTH_ACK`消息，表示这个主节点支持从节点成为新的主节点。
5.  每个参与选举的从节点都会接收`CLUSTERMSG_TYPE_FAILOVER_AUTH_ACK`消息，并根据自己收到了多少条这种消息来统计自己获得了多少主节点的支持。

<!---->

7.  如果集群里有N个具有投票权的主节点，那么当一个从节点收集到大于等于N/2+1张支持票时，这个从节点就会当选为新的主节点。
7.  因为在每一个配置纪元里面，每个具有投票权的主节点只能投一次票，所以如果有N个主节点进行投票，那么具有大于等于N/2+1张支持票的从节点只会有一个，这确保了新的主节点只会有一个。

<!---->

9.  如果在一个配置纪元里面没有从节点能收集到足够多的支持票，那么集群进入一个新的配置纪元，并再次进行选举，直到选出新的主节点为止。

这个选举新主节点的方法和第16章介绍的选举领头Sentinel 的方法非常相似，因为两者都是基于Raft算法的领头选举( leader election)方法来实现的。

# 消息（感觉不重要）

集群中的各个节点通过发送和接收消息（ message）来进行通信，我们称发送消息的节点为发送者(sender)，接收消息的节点为接收者( receiver )。节点发送的消息主要有以下五种:

-   **MEET消息: ** 当发送者接到客户端发送的`CLUSTER MEET`命令时，发送者会向接收者发送MEET消息，请求接收者加入到发送者当前所处的集群里面。
-   **PING 消息: ** 集群里的每个节点默认每隔一秒钟就会从已知节点列表中随机选出五个节点，然后对这五个节点中最长时间没有发送过PING消息的节点发送PING消息，以此来检测被选中的节点是否在线。除此之外，如果节点A最后一次收到节点B发送的PONG消息的时间，距离当前时间已经超过了节点A的`cluster-node-timeout`选项设置时长的一半，那么节点A也会向节点B发送PING消息，这可以防止节点A因为长时间没有随机选中节点B作为PING 消息的发送对象而导致对节点B的信息更新滞后。

<!---->

-   **PONG消息: ** 当接收者收到发送者发来的MEET消息或者PING消息时，为了向发送者确认这条MEET消息或者PING消息已到达，接收者会向发送者返回一条PONG消息。另外，一个节点也可以通过向集群广播自己的 PONG消息来让集群中的其他节点立即刷新关于这个节点的认识，例如当一次故障转移操作成功执行之后，新的主节点会向集群广播一条PONG消息，以此来让集群中的其他节点立即知道这个节点已经变成了主节点,并且接管了已下线节点负责的槽。
-   **FAIL消息:** 当一个主节点A判断另一个主节点B已经进人FAIL状态时，节点A会向集群广播一条关于节点B的FAIL消息，所有收到这条消息的节点都会立即将节点B标记为已下线。

<!---->

-   **PUBLISH消息:** 当节点接收到一个PUBLISH命令时，节点会执行这个命令，并向集群广播一条PUBLISH消息，所有接收到这条PUBLISH消息的节点都会执行相同的PUBLISH命令。

一条消息由消息头( header)和消息正文( data)组成，接下来的内容将首先介绍消息头，然后再分别介绍上面提到的五种不同类型的消息正文。

## 消息头

节点发送的所有消息都由一个消息头包裹，消息头除了包含消息正文之外，还记录了消

息发送者自身的一些信息，因为这些信息也会被消息接收者用到，所以严格来讲，我们可以

认为消息头本身也是消息的一部分。

每个消息头都由-一个cluster. h/ clusterMsg结构表示:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5269418d00264cc79c468074708694c9~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45522edc0f8f4ba59d7cf9c200f8c1b7~tplv-k3u1fbpfcp-zoom-1.image)

clusterMsg.data属性指向联合cluster.h/clusterMsgData，这个联合就是消息的正文:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f216ecc013945fcac8955404f7a84c3~tplv-k3u1fbpfcp-zoom-1.image)

clusterMsg结构的currentEpoch、sender、myslots等属性记录了发送者自身的节点信息，接收者会根据这些信息，在自己的clusterstate.nodes字典里找到发送者对应的clusterNode 结构，并对结构进行更新。

举个例子，通过对比接收者为发送者记录的槽指派信息，以及发送者在消息头的myslots属性记录的槽指派信息，接收者可以知道发送者的槽指派信息是否发生了变化。

又或者说，通过对比接收者为发送者记录的标识值，以及发送者在消息头的flags属性记录的标识值，接收者可以知道发送者的状态和角色是否发生了变化，例如节点状态由原来的在线变成了下线，或者由主节点变成了从节点等等。

## MEET、PING、PONG消息的实现

Redis集群中的各个节点通过Gossip协议来交换各自关于不同节点的状态信息，其中Gossip协议由MEET、PING、PONG三种消息实现，这三种消息的正文都由两个cluster.h/clusterMsgDataGossip 结构组成:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bf9c91135ae4e2faeec81a64484fc00~tplv-k3u1fbpfcp-zoom-1.image)

**那为啥这里的gossip数组大小只有1？？**

****

因为MEET、PING、PONG三种消息都使用相同的消息正文，所以节点通过消息头的type属性来判断一条消息是MEET消息、PING消息还是PONG消息。

每次发送MEET、PING、PONG消息时，发送者都从自己的已知节点列表中随机选出两个节点(可以是主节点或者从节点)，并将这两个被选中节点的信息分别保存到两个clusterMsgDataGossip 结构里面。

clusterMsgDataGossip结构记录了被选中节点的名字，发送者与被选中节点最后一次发送和接收ING消息和PONG消息的时间戳，被选中节点的地址和端口号，以及被选中节点的标识值:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3dfbda9953b848da9d19a3c8a985ccf6~tplv-k3u1fbpfcp-zoom-1.image)

当接收者收到MEET、PING、PONG消息时，接收者会访问消息正文中的两个clusterMsgDataGossip结构，并根据自己是否认识clusterMsgDataGossip结构中记录的被选中节点来选择进行哪种操作:

-   如果被选中节点不存在于接收者的已知节点列表，那么说明接收者是第一次接触到被选中节点，接收者将根据结构中记录的IP地址和端口号等信息，与被选中节点进行握手。
-   如果被选中节点已经存在于接收者的已知节点列表，那么说明接收者之前已经与被选中节点进行过接触，接收者将根据clusterMsgDataGossip 结构记录的信息，对被选中节点所对应的clusterNode 结构进行更新。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24392ed6e1d54a328742fc155c082f9e~tplv-k3u1fbpfcp-zoom-1.image)

## FAIL信息的实现

当集群里的主节点A将主节点B标记为已下线(FAIL)时，主节点A将向集群广播一条关于主节点B的FAIL消息，所有接收到这条FAIL消息的节点都会将主节点B标记为已下线。

在集群的节点数量比较大的情况下，单纯使用Gossip协议来传播节点的已下线信息会给节点的信息更新带来一定延迟，因为Gossip协议消息通常需要一段时间才能传播至整个集群，而发送FAIL消息可以让集群里的所有节点立即知道某个主节点已下线，从而尽快判断是否需要将集群标记为下线，又或者对下线主节点进行故障转移。

FAIL消息的正文由cluster.h/clusterMsgDataFail结构表示，这个结构只包含一个nodename 属性，该属性记录了已下线节点的名字:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e0046628c884634b1927cc99344c90e~tplv-k3u1fbpfcp-zoom-1.image)

因为集群里的所有节点都有一个独一无二的名字，所以FAIL消息里面只需要保存下线节点的名字，接收到消息的节点就可以根据这个名字来判断是哪个节点下线了。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56ae282805ad4fd9adf4b92d4fe5a260~tplv-k3u1fbpfcp-zoom-1.image)

## PUBLISH消息的实现

当客户端向集群中的某个节点发送命令:`PUBLISH<channel> <message>`的时候，接收到PUBLISH命令的节点不仅会向channel频道发送消息message，它还会向集群广播一条PUBLISH消息，所有接收到这条PUBLISH消息的节点都会向channel频道发送message消息。

**换句话说，向集群中的某个节点发送命令:** ` PUBLISH <channel> <message>  `**将导致集群中的所有节点都向channel频道发送message消息。**

****

举个例子，对于包含7000、7001、7002、7003四个节点的集群来说，如果节点7000收到了客户端发送的PUBLISH命令，那么节点7000将向7001、7002、7003三个节点发送PUBLISH消息,如图17-45所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89a4e0e4284a423e8d315e09d0b7159f~tplv-k3u1fbpfcp-zoom-1.image)



PUBLISH消息的正文由cluster.h/clusterMsgDataPublish结构表示:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23b2f847065844eca2ed4879fb5081a3~tplv-k3u1fbpfcp-zoom-1.image)

clusterMsgDataPublish结构的bulk_data属性是一个字节数组，这个字节数组保存了客户端通过PUBLISH命令发送给节点的channel参数和message参数，而结构的channel_len和 message_len则分别保存了channel参数的长度和message参数的长度:

-   -   其中bulk_data的0字节至channel_len-1字节保存的是channel参数。
    -   而bulk_data的channel_len字节至channel_len+message _len-1字节保存的则是message参数。



举个例子，如果节点收到的PUBLISH命令为:

PUBLISH "news.it" "hello"

那么节点发送的PUBLISH消息的clusterMsgDataPublish结构将如图17-46所示：其中 bulk_data数组的前七个字节保存了channel参数的值"news.it"，而 bulk_data数组的后五个字节则保存了message参数的值"hello"。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/046cc3b175de43fdac377a3d0389c0a8~tplv-k3u1fbpfcp-zoom-1.image)

# 总结

-   节点通过握手来将其他节点添加到自己所处的集群当中。
-   集群中的16384个槽可以分别指派给集群中的各个节点，每个节点都会记录哪些槽指派给了自己，而哪些槽又被指派给了其他节点。

<!---->

-   节点在接到一个命令请求时，会先检查这个命令请求要处理的键所在的槽是否由自己负责，如果不是的话，节点将向客户端返回一个MOVED错误，MOVED错误携带的信息可以指引客户端转向至正在负责相关槽的节点。
-   对Redis集群的重新分片工作是由redis-trib负责执行的，重新分片的关键是将属于某个槽的所有键值对从一个节点转移至另一个节点。

<!---->

-   如果节点A正在迁移槽i至节点B，那么当节点A没能在自己的数据库中找到命令指定的数据库键时，节点A会向客户端返回一个ASK错误，指引客户端到节点B继续查找指定的数据库键。
-   MOVED错误表示槽的负责权已经从一个节点转移到了另一个节点，而ASK错误只是两个节点在迁移槽的过程中使用的一种临时措施。

<!---->

-   集群里的从节点用于复制主节点，并在主节点下线时，代替主节点继续处理命令请求。
-   集群中的节点通过发送和接收消息来进行通信，常见的消息包括MEET、PING、PONG、PUBLISH、FAIL五种。


