## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 服务器中的数据库

Redis服务器将所有数据库都保存在服务器状态redis.h/redisserver结构的db数组中，db数组的每个项都是一个redis.h/redisDb结构，每个redisDb结构代表一个数据库:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/380c60730bef45e386f23e9531e7eef0~tplv-k3u1fbpfcp-zoom-1.image)

在初始化服务器时，程序会根据服务器状态的dbnum属性来决定应该创建多少个数据库:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07589961f3a44b81a084facb1daa8c79~tplv-k3u1fbpfcp-zoom-1.image)

dbnum属性的值由服务器配置的database选项决定，默认情况下，该选项的值为16，所以 Redis服务器默认会创建16个数据库，如图9-1所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2bc0fe4b64e74ab1b36ac5111444700d~tplv-k3u1fbpfcp-zoom-1.image)

# 切换数据库

每个Redis客户端都有自己的目标数据库，每当客户端执行数据库写命令或者数据库读命令的时候,目标数据库就会成为这些命令的操作对象。

默认情况下，Redis客户端的目标数据库为0号数据库，但客户端可以通过执行`SELECT`命令来切换目标数据库。如：`SELECT 1`切换到1号数据库

在服务器内部，客户端状态redisclient结构的db属性记录了客户端当前的目标数据库，这个属性是一个指向redisDb 结构的指针:

```
typedef struct redisClient {
    // ... . ..
    
	//记录客户端当前正在使用的数据库
    redisDb *db;
    
	//...
    
}redisclient;
```

redisclient.db 指针指向redisServer.db数组的其中一个元素，而被指向的元素就是客户端的目标数据库。

比如说，如果某个客户端的目标数据库为1号数据库，那么这个客户端所对应的客户端状态和服务器状态之间的关系如图9-2所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/887dd53090ee49879c63b252bf143573~tplv-k3u1fbpfcp-zoom-1.image)

切换数据库就是让db指针指向其他元素

# 数据库键空间

Redis是一个键值对( key-value pair)数据库服务器，服务器中的每个数据库都由一个redis.h/redisDb结构表示，其中，redisDb结构的dict字典保存了数据库中的所有键值对，我们将这个字典称为键空间( key space ) :

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98dae913d7ca47b793be44f66236c223~tplv-k3u1fbpfcp-zoom-1.image)

键空间和用户所见的数据库是直接对应的:

-   键空间的键也就是数据库的键，每个键都是一个字符串对象。
-   键空间的值也就是数据库的值，每个值可以是字符串对象、列表对象、哈希表对象、集合对象和有序集合对象中的任意一种 Redis对象。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e7f384988f148c6a37f70cf6512288c~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b41da30c1c43410ba6ed7f6912f1ffd2~tplv-k3u1fbpfcp-zoom-1.image)

## 读写键空间时的维护操作

当使用Redis命令对数据库进行读写时，服务器不仅会对键空间执行指定的读写操作，还会执行一些额外的维护操作，其中包括:

-   在读取一个键之后（读操作和写操作都要对键进行读取)，服务器会根据键是否存在来更新服务器的键空间命中( hit）次数或键空间不命中(miss)次数，这两个值可以在 INFO stats命令的keyspace_hits属性和 keyspace_misses属性中查看。
-   在读取一个键之后，服务器会更新键的LRU(最后一次使用)时间，这个值可以用于计算键的闲置时间，使用`OBJECT idletime <key>`命令可以查看键key的闲置时间。

<!---->

-   如果服务器在读取一个键时发现该键已经过期，那么服务器会先删除这个过期键，然后才执行余下的其他操作。
-   如果有客户端使用`WATCH`命令监视了某个键，那么服务器在对被监视的键进行修改之后，会将这个键标记为脏（ dirty )，从而让事务程序注意到这个键已经被修改过。

<!---->

-   服务器每次修改一个键之后，都会对脏( dirty)键计数器的值增1，这个计数器会触发服务器的持久化以及复制操作。
-   如果服务器开启了数据库通知功能，那么在对键进行修改之后，服务器将按配置发送相应的数据库通知。

# AOF，RDB和复制功能对过期键的处理

## 生成RDB文件

在执行SAVE命令或者BGSAVE命令创建一个新的RDB文件时，程序会对数据库中的键进行检查，已过期的键不会被保存到新创建的RDB 文件中。

举个例子，如果数据库中包含三个键k1、k2、k3，并且k2已经过期，那么当执行SAVE命令或者BGSAVE命令时，程序只会将k1和k3的数据保存到RDB文件中，而k2则会被忽略。

**因此，数据库中包含过期键不会对生成新的RDB文件造成影响。**

## 载入RDB文件

在启动Redis服务器时，如果服务器开启了RDB 功能，那么服务器将对RDB文件进行载入:

-   如果服务器以主服务器模式运行，那么在载人RDB文件时，程序会对文件中保存的键进行检查，未过期的键会被载入到数据库中，而过期键则会被忽略，所以过期键对载入RDB文件的主服务器不会造成影响。
-   如果服务器以从服务器模式运行，那么在载入RDB文件时，文件中保存的所有键，不论是否过期，都会被载入到数据库中。不过，因为主从服务器在进行数据同步的时候，从服务器的数据库就会被清空，所以一般来讲，过期键对载入RDB文件的从服务器也不会造成影响。

## AOF文件写入

当服务器以AOF持久化模式运行时，如果数据库中的某个键已经过期，但它还没有被惰性删除或者定期删除，那么AOF 文件不会因为这个过期键而产生任何影响。

当过期键被惰性删除或者定期删除之后，程序会向AOF文件追加（ append)一条DEL命令，来显式地记录该键已被删除。

举个例子，如果客户端使用GET message命令，试图访问过期的message键，那么服务器将执行以下三个动作:

1.  从数据库中删除message 键。
1.  追加一条DEL message命令到AOF文件。

<!---->

3.  向执行GET命令的客户端返回空回复。

## AOF文件重写

和生成RDB文件时类似，在执行AOF重写的过程中，程序会对数据库中的键进行检查，已过期的键不会被保存到重写后的AOF文件中。

举个例子，如果数据库中包含三个键k1、k2、k3，并且 k2已经过期，那么在进行重写工作时，程序只会对k1和k3进行重写，而k2则会被忽略。

因此，数据库中包含过期键不会对AOF重写造成影响。

## 复制

当服务器运行在复制模式下时，从服务器的过期键删除动作由主服务器控制:

-   主服务器在删除一个过期键之后，会显式地向所有从服务器发送一个DEL命令，告知从服务器删除这个过期键。
-   从服务器在执行客户端发送的读命令时，即使碰到过期键也不会将过期键删除，而是继续像处理未过期的键一样来处理过期键。**所以此时读取时，客户端会读到过期的值。**

<!---->

-   从服务器只有在接到主服务器发来的DEL命令之后，才会删除过期键。

通过由主服务器来控制从服务器统一地删除过期键，可以保证主从服务器数据的一致性，也正是因为这个原因，当一个过期键仍然存在于主服务器的数据库时，这个过期键在从服务器里的复制品也会继续存在。

# 数据库通知

数据库通知是Redis 2.8版本新增加的功能，这个功能可以让客户端通过订阅给定的频道或者模式，来获知数据库中键的变化，以及数据库中命令的执行情况。

举个例子，以下代码展示了客户端如何获取0号数据库中针对message键执行的所有命令:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18a65f0592cf4c54a595383029ac98e8~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a5f4134311641d2ac63da7ceebf1a41~tplv-k3u1fbpfcp-zoom-1.image)

根据发回的通知显示，先后共有SET、EXPIRE、DEL三个命令对键message进行了操作。

这一类关注“某个键执行了什么命令”的通知称为**键空间通知( key-space notification )** ,除此之外，还有另一类称为**键事件通知( key-event notification** )的通知，**它们关注的是“某个命令被什么键执行了”** 。

以下是一个键事件通知的例子，代码展示了客户端如何获取0号数据库中所有执行DEL命令的键:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/053ea1e3c1de4a5ba403ae5d96c10853~tplv-k3u1fbpfcp-zoom-1.image)

服务器配置的notify-keyspace-events选项决定了服务器所发送通知的类型;

-   想让服务器发送所有类型的键空间通知和键事件通知，可以将选项的值设置为AKE。
-   想让服务器发送所有类型的键空间通知，可以将选项的值设置为AK。

<!---->

-   想让服务器发送所有类型的键事件通知，可以将选项的值设置为AE。
-   想让服务器只发送和字符串键有关的键空间通知，可以将选项的值设置为K$。

<!---->

-   想让服务器只发送和列表键有关的键事件通知，可以将选项的值设置为El。

## 原理

发送数据库通知的功能是由`notify.c/notifyKeyspaceEvent`函数实现的:

`void notifyKeyspaceEvent(int type, char *event,robj *key,int dbid);`

函数的type参数是当前想要发送的通知的类型，程序会根据这个值来判断通知是否就是服务器配置

`notify-keyspace-events`选项所选定的通知类型，从而决定是否发送通知。

`event` 、`keys`和 `dbid`分别是事件的名称、产生事件的键，以及产生事件的数据库号码，函数会根据`type`参数以及这三个参数来构建事件通知的内容，以及接收通知的频道名。

每当一个 Redis命令需要发送数据库通知的时候，该命令的实现函数就会调用`notify-KeyspaceEvent`函数，并向函数传递传递该命令所引发的事件的相关信息。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f79a6c4912f54c4fa3c6aaace8c93c35~tplv-k3u1fbpfcp-zoom-1.image)

当SADD命令至少成功地向集合添加了一个集合元素之后，命令就会发送通知，该通知的类型为`REDIS_NOTIFY_SET`(表示这是一个集合键通知)，名称为sadd(表示这是执行SADD命令所产生的通知)。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70971d0664a240548a3e14a60a0787f3~tplv-k3u1fbpfcp-zoom-1.image)

# 总结

-   Redis服务器的所有数据库都保存在redisserver.db数组中，而数据库的数量则由redisserver.dbnum属性保存。
-   客户端通过修改目标数据库指针，让它指向redisServer.db数组中的不同元素来切换不同的数据库。

<!---->

-   数据库主要由dict和expires两个字典构成，其中dict字典负责保存键值对，而expires字典则负责保存键的过期时间。
-   因为数据库由字典构成，所以对数据库的操作都是建立在字典操作之上的。

<!---->

-   数据库的键总是一个字符串对象，而值则可以是任意一种Redis对象类型，包括字符串对象、哈希表对象、集合对象、列表对象和有序集合对象，分别对应字符串键、哈希表键、集合键、列表键和有序集合键。
-   expires字典的键指向数据库中的某个键，而值则记录了数据库键的过期时间，过期时间是一个以毫秒为单位的UNIX时间戳。

<!---->

-   Redis使用惰性删除和定期删除两种策略来删除过期的键：惰性删除策略只在碰到过期键时才进行删除操作，定期删除策略则每隔一段时间主动查找并删除过期键。执行SAVE命令或者BGSAVE命令所产生的新RDB文件不会包含已经过期的键。执行BGREWRITEAOF命令所产生的重写AOF文件不会包含已经过期的键。
-   当一个过期键被删除之后，服务器会追加一条DEL命令到现有AOF文件的末尾，显式地删除过期键。

<!---->

-   当主服务器删除一个过期键之后，它会向所有从服务器发送一条DEL命令，显式地删除过期键。
-   从服务器即使发现过期键也不会自作主张地删除它，而是等待主节点发来DEL命令，这种统一、中心化的过期键删除策略可以保证主从服务器数据的一致性。

<!---->

-   当Redis命令对数据库进行修改之后，服务器会根据配置向客户端发送数据库通知。

其他内容比如键空间的增删改查，过期，删除策略等等不再赘述，和“使用篇”相比没有深入太多

