---
theme: qklhk-chocolate
---
对于每个与服务器进行连接的客户端，服务器都为这些客户端建立了相应的redis.h/redisclient结构（客户端状态)，这个结构保存了客户端当前的状态信息，以及执行相关功能时需要用到的数据结构，其中包括：

-   -   客户端的套接字描述符。
    -   客户端的名字。

<!---->

-   -   客户端的标志值（ flag )。
    -   指向客户端正在使用的数据库的指针，以及该数据库的号码。

<!---->

-   -   客户端当前要执行的命令、命令的参数、命令参数的个数，以及指向命令实现函数的指针。
    -   客户端的输人缓冲区和输出缓冲区。

<!---->

-   -   客户端的复制状态信息，以及进行复制所需的数据结构。
    -   客户端执行BRPOP、BLPOP等列表阻塞命令时使用的数据结构。

<!---->

-   -   客户端的事务状态，以及执行WATCH命令时用到的数据结构。
    -   客户端执行发布与订阅功能时用到的数据结构。

<!---->

-   -   客户端的身份验证标志。
    -   客户端的创建时间，客户端和服务器最后一次通信的时间，以及客户端的输出缓冲区大小超出软性限制（ soft limit）的时间。

Redis服务器状态结构的clients属性是一个链表，这个链表保存了所有与服务器连接的客户端的状态结构，对客户端执行批量操作，或者查找某个指定的客户端，都可以通过遍历clients链表来完成:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28514d21c68d4d2e9c8519d5b5c538fc~tplv-k3u1fbpfcp-zoom-1.image)

# 客户端属性

客户端状态包含的属性可以分为两类:

-   一类是比较通用的属性，这些属性很少与特定功能相关，无论客户端执行的是什么工作，它们都要用到这些属性。
-   另外一类是和特定功能相关的属性，比如操作数据库时需要用到的db属性和dictid属性，执行事务时需要用到的mstate属性，以及执行WATCH命令时需要用到的watched_keys属性等等。

## 套接字描述符

客户端状态的fd属性记录了客户端正在使用的套接字描述符。根据客户端类型的不同，fd属性的值可以是-1或者是大于-1的整数:

-   **伪客户端**( fake client)的fd属性的值为-1:伪客户端处理的命令请求来源于AOF文件或者Lua脚本，而不是网络，所以这种客户端不需要套接字连接，自然也不需要记录套接字描述符。目前Redis服务器会在两个地方用到伪客户端，一个用于载人AOF文件并还原数据库状态，而另一个则用于执行Lua脚本中包含的Redis命令。
-   **普通客户端**的fd属性的值为大于-1的整数：普通客户端使用套接字来与服务器进行通信，所以服务器会用fd属性来记录客户端套接字的描述符。因为合法的套接字描述符不能是–1，所以普通客户端的套接字描述符的值必然是大于-1的整数。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d1f335a1bed4dfb9fd2e8a3fe1595fa~tplv-k3u1fbpfcp-zoom-1.image)

## 名字

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b80c240dd5b94e7f8fe8fdb8e48eacd5~tplv-k3u1fbpfcp-zoom-1.image)

## 标志

客户端的标志属性flags记录了客户端的角色（role )，以及客户端目前所处的状态:

flags属性的值可以是单个标志:

flags = <flag>

也可以是多个标志的二进制或，比如:

flags = <flag1> l <flag2> l ...

每个标志使用一个常量表示，一部分标志记录了客户端的角色：

-   在主从服务器进行复制操作时，主服务器会成为从服务器的客户端，而从服务器也会成为主服务器的客户端。`REDIS_MASTER`标志表示客户端代表的是一个主服务器，REDIS_SLAVE标志表示客户端代表的是一个从服务器。
-   `REDIS_PRE_PSYNC`标志表示客户端代表的是一个版本低于Redis2.8的从服务器，主服务器不能使用PSYNC命令与这个从服务器进行同步。这个标志只能在REDIS_SLAVE标志处于打开状态时使用。

<!---->

-   `REDIS_LUA_CLIENT`标识表示客户端是专门用于处理Lua脚本里面包含的Redis命令的伪客户端。

而另外一部分标志则记录了客户端目前所处的状态:

-   `REDIS_MONITOR`标志表示客户端正在执行MONITOR命令。
-   `REDIS_UNIX_SOCKET`标志表示服务器使用UNIX套接字来连接客户端。

<!---->

-   `REDIS_BLOCKED`标志表示客户端正在被BRPOP、BLPOP等命令阻塞。

更多标志都定义在redis.h文件里面。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3b0f03203894cf5b148d98508165c48~tplv-k3u1fbpfcp-zoom-1.image)

## 输入缓冲区

客户端状态的输入缓冲区用于保存客户端发送的命令请求。

举个例子，如果客户端向服务器发送了以下命令请求:

SET key value

那么客户端状态的querybuf属性将是一个包含以下内容的SDS值:

* 3 \r\n$3\r\nSET \r\n$ 3lrinkey\r\n$5\r\nvalue\r\n

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5b417df134142a393a2e15e8940b087~tplv-k3u1fbpfcp-zoom-1.image)

输人缓冲区的大小会根据输人内容动态地缩小或者扩大，但它的最大大小不能超过1GB，否则服务器将关闭这个客户端。

## 命令与命令参数

在服务器将客户端发送的命令请求保存到客户端状态的querybuf属性之后，服务器将对命令请求的内容进行分析，并将得出的命令参数以及命令参数的个数分别保存到客户端状态的argv属性和argc属性；

**argv属性是一个数组，数组中的每个项都是一个字符串对象，其中argv[0]是要执行的命令，而之后的其他项则是传给命令的参数。**

**argc属性则负责记录argv数组的长度。**

举个例子，对于图13-4所示的querybuf属性来说，服务器将分析并创建图13-5所示的argv属性和argc属性。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24d8747e706e49439e67249525e4b417~tplv-k3u1fbpfcp-zoom-1.image)

## 命令的实现函数

当服务器从协议内容中分析并得出argv属性和argc属性的值之后，服务器将根据项argv[0]的值，在命令表中查找命令所对应的命令实现函数。

图13-6展示了一个命令表示例，该表是一个字典，字典的键是一个SDS结构，保存了命令的名字，字典的值是命令所对应的redisCommand结构，这个结构保存了命令的实现函数、命令的标志、命令应该给定的参数个数、命令的总执行次数和总消耗时长等统计信息。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72db8829e05f425d83e12216a3e10996~tplv-k3u1fbpfcp-zoom-1.image)

当程序在命令表中成功找到argv[0]所对应的redisCommand结构时,它会将客户端状态的cmd指针指向这个结构:

之后，服务器就可以使用cmd属性所指向的rediscommand结构，以及argv、argc属性中保存的命令参数信息，调用命令实现函数，执行客户端指定的命令。

图13-7演示了服务器在argv[0]为"SET"时，查找命令表并将客户端状态的cmd指针指向目标rediscommand结构的整个过程。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62f88e94667f499c947cb6c797f65370~tplv-k3u1fbpfcp-zoom-1.image)

## 输出缓冲区

执行命令所得的命令回复会被保存在客户端状态的输出缓冲区里面，每个客户端都有两个输出缓冲区可用，一个缓冲区的大小是固定的，另一个缓冲区的大小是可变的:

-   -   固定大小的缓冲区用于保存那些长度比较小的回复，比如oK、简短的字符串值、整数值、错误回复等等。
    -   可变大小的缓冲区用于保存那些长度比较大的回复，比如一个非常长的字符串值，一个由很多项组成的列表，一个包含了很多元素的集合等等。

客户端的固定大小缓冲区由buf和 bufpos两个属性组成:

-   -   buf是一个大小为REDIS_REPLY_CHUNK_BYTES字节的字节数组
    -   而bufpos属性则记录了buf数组目前已使用的字节数量。

REDIS REPLY_CHUNK_BYTES常量目前的默认值为16*1024，也即是说，buf数组的默认大小为16KB。

图13-8展示了一个使用固定大小缓冲区来保存返回值+OK\r\n的例子。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ae3bdecabc64953abb9998dd0ee29d8~tplv-k3u1fbpfcp-zoom-1.image)

当buf数组的空间已经用完，或者回复因为太大而没办法放进buf数组里面时，服务器就会开始使用可变大小缓冲区。可变大小缓冲区由reply链表和一个或多个字符串对象组成:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b30e89c561dd4bf7ab68f5fb82fefbf8~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/101b2d055df349e1b1e9923e9f5c9d18~tplv-k3u1fbpfcp-zoom-1.image)

## 身份验证

客户端状态的authenticated属性用于记录客户端是否通过了身份验证。如果authenticated的值为0，那么表示客户端未通过身份验证;如果authenticated的值为1，那么表示客户端已经通过了身份验证。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19a10b9a601c4f47a2b9d4d1e3708d90~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dcd57e2001564a58b1abfb742ecf7536~tplv-k3u1fbpfcp-zoom-1.image)

当客户端authenticated属性的值为0时，除了AUTH命令之外，客户端发送的所有其他命令都会被服务器拒绝执行:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83f8a156c25d4a988f838c77a66df68b~tplv-k3u1fbpfcp-zoom-1.image)

authenticated属性仅在服务器启用了身份验证功能时使用。如果服务器没有启用身份验证功能的话，那么即使authenticated属性的值为0（这是默认值)，服务器也不会拒绝执行客户端发送的命令请求。

关于服务器身份验证的更多信息可以参考示例配置文件对requirepass 选项的相关说明。

## 时间

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d1c53cd441c4512bab742d304d53ee2~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea071e83f8aa48538451c2b899045b75~tplv-k3u1fbpfcp-zoom-1.image)

# 客户端的创建与关闭

## 创建普通客户端

如果客户端是通过网络连接与服务器进行连接的普通客户端，那么在客户端使用connect函数连接到服务器时，服务器就会调用连接事件处理器，为客户端创建相应的客户端状态，并将这个新的客户端状态添加到服务器状态结构clients链表的末尾。

举个例子，假设当前有cl和 c2两个普通客户端正在连接服务器，那么当一个新的普通客户端c3连接到服务器之后，服务器会将c3所对应的客户端状态添加到clients链表的末尾，如图13-12所示，其中用虚线包围的就是服务器为c3新创建的客户端状态。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f28fd6d263ba47e7921cf49cb4682b08~tplv-k3u1fbpfcp-zoom-1.image)

## 关闭普通客户端

关闭一个普通客户端的原因很多：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/749d9f0c806d48d2b45ad0bc1eccab18~tplv-k3u1fbpfcp-zoom-1.image)

前面介绍输出缓冲区的时候提到过，可变大小缓冲区由一个链表和任意多个字符串对象组成，理论上来说，这个缓冲区可以保存任意长的命令回复。

但是，为了避免客户端的回复过大，占用过多的服务器资源，服务器会时刻检查客户端的输出缓冲区的大小，并在缓冲区的大小超出范围时，执行相应的限制操作。

服务器使用两种模式来限制客户端输出缓冲区的大小:

-   -   硬性限制( hard limit ) :如果输出缓冲区的大小超过了硬性限制所设置的大小，那么服务器立即关闭客户端。
    -   软性限制（ soft limit ) :如果输出缓冲区的大小超过了软性限制所设置的大小，但还没超过硬性限制，那么服务器将使用客户端状态结构的obuf_soft_limitreached_time属性记录下客户端到达软性限制的起始时间；之后服务器会继续监视客户端，如果输出缓冲区的大小一直超出软性限制，并且持续时间超过服务器设定的时长，那么服务器将关闭客户端;相反地，如果输出缓冲区的大小在指定时间之内，不再超出软性限制，那么客户端就不会被关闭，并且 obuf_soft_limit_reached_time属性的值也会被清零。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0c1535479ca457da2e92ae26aca5243~tplv-k3u1fbpfcp-zoom-1.image)

## Lua脚本的伪客户端

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2c64bd24f61498fb45cc8618ca4f8b8~tplv-k3u1fbpfcp-zoom-1.image)

## AOF载入的伪客户端

服务器在载入AOF文件时，会创建用于执行AOF文件包含的Redis命令的伪客户端，并在载入完成之后，关闭这个伪客户端。

# 总结

-   -   服务器状态结构使用clients链表连接起多个客户端状态，新添加的客户端状态会被放到链表的末尾。
    -   客户端状态的flags属性使用不同标志来表示客户端的角色，以及客户端当前所处的状态。

<!---->

-   -   输入缓冲区记录了客户端发送的命令请求，这个缓冲区的大小不能超过1GB.
    -   命令的参数和参数个数会被记录在客户端状态的argv和argc属性里面，而cmd属性则记录了客户端要执行命令的实现函数。

<!---->

-   -   客户端有固定大小缓冲区和可变大小缓冲区两种缓冲区可用，其中固定大小缓冲区的最大大小为16 KB，而可变大小缓冲区的最大大小不能超过服务器设置的硬性限制值。
    -   输出缓冲区限制值有两种，如果输出缓冲区的大小超过了服务器设置的硬性限制，那么客户端会被立即关闭；除此之外，如果客户端在一定时间内，一直超过服务器设置的软性限制，那么客户端也会被关闭。

<!---->

-   -   当一个客户端通过网络连接连上服务器时，服务器会为这个客户端创建相应的客户端状态。网络连接关闭、发送了不合协议格式的命令请求、成为CLIENT KILL命令的目标、空转时间超时、输出缓冲区的大小超出限制，以上这些原因都会造成客户端被关闭。
    -   处理Lua脚本的伪客户端在服务器初始化时创建，这个客户端会一直存在，直到服务器关闭。

<!---->

-   -   载人AOF 文件时使用的伪客户端在载人工作开始时动态创建，载入工作完毕之后关闭。


