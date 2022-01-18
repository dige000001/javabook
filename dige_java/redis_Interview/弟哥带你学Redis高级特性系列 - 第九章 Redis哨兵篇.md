## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

本节主要展开故障转移的整个过程，其他内容和简略见之前文章

# 启动并初始化sentinel

redis-sentinel /path/to/your/ sentinel.conf

## 初始化服务器

首先，因为Sentinel本质上只是一个运行在特殊模式下的Redis服务器，所以启动Sentinel的第一步，就是初始化一个普通的Redis服务器

不过，因为Sentinel执行的工作和普通Redis服务器执行的工作不同，所以Sentinel 的初始化过程和普通Redis服务器的初始化过程并不完全相同。

例如，普通服务器在初始化时会通过载入RDB文件或者AOF文件来还原数据库状态，但是因为Sentinel并不使用数据库，所以初始化Sentinel时就不会载入RDB文件或者AOF文件。

表16-1展示了Redis服务器在Sentinel模式下运行时，服务器各个主要功能的使用情况。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/001791c6f1d947f6ab75de7251b86a60~tplv-k3u1fbpfcp-zoom-1.image)

## 使用sentinel专用代码

启动Sentinel的第二个步骤就是将一部分普通Redis服务器使用的代码替换成Sentinel专用代码。如Sentinel则使用sentinel.c/REDIS_SENTINEL_PORT常量的值作为服务器端口

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fc2f29dfe5140508deae503f46de13c~tplv-k3u1fbpfcp-zoom-1.image)

## 初始化sentinel状态

在应用了Sentinel的专用代码之后，接下来，服务器会初始化一个sentinel.c/sentinelstate结构（后面简称“Sentinel状态”)，这个结构保存了服务器中所有和Sentinel功能有关的状态（服务器的一般状态仍然由redis.h/redisServer结构保存):

```
struct sentinelState {
    
	//当前纪元，用于实现故障转移
	uint64_t current_epoch;
    
	//保存了所有被这个sentine1监视的主服务器
    //字典的键是主服务器的名字
	//字典的值则是一个指向sentinelRedisInstance结构的指针
    dict *masters;
    
	//是否进入了TILT模式?
    int tilt;
    
	//目前正在执行的脚本的数量
    int running_scripts;
    
    //进入TILT模式的时间
	mstime_t tilt_start_time;
    
    //最后一次执行时间处理器的时间
    mstime_t previous_time;
    
	//一个FIFO队列，包含了所有需要执行的用户脚本
    list *scripts_queue;
    
}sentinel;
```

## 初始化 Sentinel状态的masters属性

Sentinel 状态中的masters字典记录了所有被Sentinel 监视的主服务器的相关信息，其中:

-   -   字典的键是被监视主服务器的名字。
    -   而字典的值则是被监视主服务器对应的sentinel.c/**sentinelRedisInstance**结构。

每个sentinelRedisInstance结构（后面简称“实例结构”)代表一个被Sentinel监视的Redis服务器实例(instance)，这个实例可以是主服务器、从服务器，**或者另外一个Sentinel**。

实例结构包含的属性非常多，以下代码展示了实例结构在表示主服务器时使用的其中一部分属性，本章接下来将逐步对实例结构中的各个属性进行介绍:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34bdb03a909840c7899760f0df20cc7f~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9840ee84fca640e98aafed6beac5ce76~tplv-k3u1fbpfcp-zoom-1.image)

sentinelRedisInstance.addr属性是一个指向sentinel.c/sentinelAddr结构的指针，这个结构保存着实例的IP地址和端口号:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9237e5b0192346a0a332d8f3d4907349~tplv-k3u1fbpfcp-zoom-1.image)

对Sentinel 状态的初始化将引发对masters字典的初始化，而masters字典的初始化是根据被载人的Sentinel配置文件来进行的。

举个例子，如果用户在启动Sentinel时，指定了包含以下内容的配置文件:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4bed8496ae6c49569a741edf6932c9db~tplv-k3u1fbpfcp-zoom-1.image)

那么Sentinel将为主服务器masterl创建如图16-5所示的实例结构，并为主服务器master2创建如图16-6所示的实例结构，而这两个实例结构又会被保存到Sentinel状态的masters字典中，如图16-7所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d2dec3b993349de8ffea1e5ce636d86~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a4232a4bbfb4cc5a7ff69ae1e58bfce~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5392ef985f2b47f39f2e70cd89871432~tplv-k3u1fbpfcp-zoom-1.image)

## 创建连向主服务器的网络连接

初始化Sentinel的最后一步是创建连向被监视主服务器的网络连接，Sentinel将成为主服务器的客户端，它可以向主服务器发送命令，并从命令回复中获取相关的信息。

对于每个被Sentinel监视的主服务器来说，Sentinel会创建两个连向主服务器的异步网络连接:

-   一个是命令连接，这个连接专门用于向主服务器发送命令，并接收命令回复。
-   另一个是订阅连接，这个连接专门用于订阅主服务器的_sentinel_:hello频道。

# 获取主服务器信息

Sentinel默认会以每十秒一次的频率，通过命令连接向被监视的主服务器发送INFO命令，并通过分析INFO命令的回复来获取主服务器的当前信息。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c835dd39a0ae424ab047f088f6af97f4~tplv-k3u1fbpfcp-zoom-1.image)

**主要获得主服务器的运行id和role域的运行角色，从服务器的ip端口和offset**

根据run_id域和role域记录的信息，Sentinel将对主服务器的实例结构进行更新，例如，主服务器重启之后，它的运行ID就会和实例结构之前保存的运行ID不同，Sentinel检测到这一情况之后，就会对实例结构的运行ID进行更新。

至于主服务器返回的从服务器信息，则会被用于更新主服务器实例结构的slaves字典，这个字典记录了主服务器属下从服务器的名单:

-   -   字典的键是由Sentinel自动设置的从服务器名字，格式为ip:port:如对于IP地址为127.0.0.1，端口号为11111的从服务器来说，Sentinel为它设置的名字就是127.0.0.1 : 11111。
    -   至于字典的值则是从服务器对应的实例结构:比如说，如果键是127.0.0.1:11111,那么这个键的值就是P地址为127.0.0.1，端口号为11111的从服务器的实例结构。

Sentinel在分析INFO命令中包含的从服务器信息时，会检查从服务器对应的实例结构是否已经存在于slaves 字典：若存在则更新，否则创建新的实例

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a31611ce08b54cca995fcf5011366365~tplv-k3u1fbpfcp-zoom-1.image)

注意对比图中主服务器实例结构和从服务器实例结构之间的区别:

-   -   主服务器实例结构的flags属性的值为SRT_MASTER，而从服务器实例结构的flags属性的值为SRI_SLAVE。
    -   主服务器实例结构的name属性的值是用户使用Sentinel配置文件设置的，而从服务器实例结构的name属性的值则是Sentinel根据从服务器的IP地址和端口号自动设置的。

# 获取从服务器信息

当Sentinel发现主服务器有新的从服务器出现时，Sentinel除了会为这个新的从服务器创建相应的实例结构之外，Sentinel还会创建连接到从服务器的命令连接和订阅连接。

在创建命令连接之后，Sentinel在默认情况下，会以每十秒一次的频率通过命令连接向从服务器发送 INFO命令，并获得类似于以下内容的回复:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/324eb9a7e8664ea0930aedea950cb78f~tplv-k3u1fbpfcp-zoom-1.image)

根据INFO命令的回复，Sentinel会提取出以下信息:

-   -   从服务器的运行ID run_id。
    -   从服务器的角色role。

<!---->

-   -   主服务器的IP地址master_host，以及主服务器的端口号master_port。
    -   主从服务器的连接状态master_link_status。

<!---->

-   -   从服务器的优先级slave_priorityo
    -   从服务器的复制偏移量slave_repl_offset。

根据这些信息，Sentinel会对从服务器的实例结构进行更新，图16-12展示了Sentinel根据上面的INFO命令回复对从服务器的实例结构进行更新之后，实例结构的样子。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f37a281852c046c2a57f0550a16faf46~tplv-k3u1fbpfcp-zoom-1.image)

# 向主服务器和从服务器发送信息

在默认情况下，Sentinel 会以每2秒一次的频率，通过命令连接向所有被监视的主服务器和从服务器发送以下格式的命令:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e034b37fa9c7434dac326a9c5dba1c85~tplv-k3u1fbpfcp-zoom-1.image)

这条命令向服务器的__sentinel__:hello频道发送了一条信息，信息的内容由多个参数组成:

-   其中以s_开头的参数记录的是Sentinel本身的信息，各个参数的意义如表16-2所示。
-   而m开头的参数记录的则是主服务器的信息，各个参数的意义如表16-3所示。如果Sentinel正在监视的是主服务器，那么这些参数记录的就是主服务器的信息；如果Sentinel正在监视的是从服务器，那么这些参数记录的就是从服务器正在复制的主服务器的信息。

**注意该信息会被频道里主服务器和所有监控此频道的sentinel获取**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8a9e8b868324a669a19080dab38bb78~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc60f9df9ff0409a8fe8ed0da4ed629b~tplv-k3u1fbpfcp-zoom-1.image)

# 接收来自主服务器和从服务器的频道信息

当Sentinel与一个主服务器或者从服务器建立起订阅连接之后，Sentinel就会通过订阅连接，向服务器发送以下命令:

SUBSCRIBE_sentinel_:hello

Sentinel对_sentinel_:hello频道的订阅会一直持续到Sentinel 与服务器的连接断开为止。

对于监视同一个服务器的多个Sentinel来说，一个Sentinel发送的信息会被其他Sentinel接收到，这些信息会被用于更新其他Sentinel对发送信息Sentinel的认知,也会被用于更新其他 Sentinel对被监视服务器的认知。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a4552632df24a22ad3c0f8879e5b1cb~tplv-k3u1fbpfcp-zoom-1.image)

## 更新sentinels字典

Sentinel为主服务器创建的实例结构（redisSentinelInstance）中的sentinels字典保存了除Sentinel本身之外，所有同样监视这个主服务器的其他Sentinel的资料:

-   sentinels字典的键是其中一个Sentinel的名字，格式为ip:port，比如对于IP地址为127.0.0.1，端口号为26379的Sentinel来说，这个Sentinel在sentinels字典中的键就是"127.0.0.1 :26379"。
-   sentinels字典的值则是键所对应 Sentinel的实例结构，比如对于键"127.0.0.1:26379"来说，这个键在sentinels字典中的值就是IP为127.0.0.1，端口号为26379的 Sentinel的实例结构（也是redisSentinelInstance，没想到吧）。

当一个Sentinel接收到其他Sentinel发来的信息时（我们称呼发送信息的Sentinel为源Sentinel，接收信息的Sentinel为目标Sentinel )，目标Sentinel会从信息中分析并提取出以下两方面参数:

-   与Sentinel有关的参数:源Sentinel的IP地址、端口号、运行ID和配置纪元。
-   与主服务器有关的参数:源Sentinel正在监视的主服务器的名字、IP地址、端口号和配置纪元。

根据信息中提取出的主服务器参数，目标Sentinel会在自己的Sentinel状态的masters字典中查找相应的主服务器实例结构，然后根据提取出的Sentinel参数，检查主服务器实例结构的sentinels字典中，源Sentinel的实例结构是否存在，若存在则更新，否则创建新实例结构。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54dc370dd049400ab4a0156e7911104b~tplv-k3u1fbpfcp-zoom-1.image)

因为一个Sentinel可以通过分析接收到的频道信息来获知其他Sentinel的存在，并通过发送频道信息来让其他Sentinel知道自己的存在，所以用户在使用Sentinel的时候并不需要提供各个Sentinel的地址信息，监视同一个主服务器的多个Sentinel可以自动发现对方。

## 创建连向其他Sentinel的命令连接

当Sentinel通过频道信息发现一个新的Sentinel时，它不仅会为新Sentinel在 sentinels 字典中创建相应的实例结构，**还会创建一个连向新Sentinel的命令连接，而新Sentinel也同样会创建连向这个Sentinel 的命令连接，最终监视同一主服务器的多个Sentinel将形成相互连接的网络：Sentinel A有连向Sentinel B的命令连接，而Sentinel B也有连向Sentinel A的命令连接。**

图16-16展示了三个监视同一主服务器的Sentinel之间是如何互相连接的。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/049e30ca522b4615bfd9dac515d05ede~tplv-k3u1fbpfcp-zoom-1.image)

使用命令连接相连的各个Sentinel可以通过向其他Sentinel 发送命令请求来进行信息交换

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb9a5cd13227444d9513e9221800025f~tplv-k3u1fbpfcp-zoom-1.image)

# 检测主观下线状态

在默认情况下，Sentinel会以每秒一次的频率向所有与它创建了命令连接的实例(包括主服务器、从服务器、其他Sentinel在内）发送PING命令，并通过实例返回的PING命令回复来判断实例是否在线。

在图16-17展示的例子中，带箭头的连线显示了Sentinel1和 Sentinel2是如何向实例发送PING命令的:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1e3e56681a04f81a5414f8ae19a5b14~tplv-k3u1fbpfcp-zoom-1.image)

-   Sentinel1将向Sentinel2、主服务器master 、从服务器slave1和 slave2发送 PING命令。
-   Sentinel2将向Sentinel1、主服务器master 、从服务器slave1和 slave2发送 PING命令。

实例对PING命令的回复可以分为以下两种情况:

-   有效回复:实例返回+PONG、-LOADING、-MASTERDOWN三种回复的其中一种。
-   无效回复:实例返回除+PONG、-LOADING、一MASTERDOWN三种回复之外的其他回复，或者在指定时限内没有返回任何回复。

Sentinel配置文件中的down-after-milliseconds选项指定了Sentine判断实例进入主观下线所需的时间长度：如果一个实例在down-after-milliseconds毫秒内，连续向Sentinel返回无效回复，那么Sentinel会修改这个实例所对应的实例结构，在结构的flags属性中打开SRT_S_DOWN标识，以此来表示这个实例已经进人主观下线状态。

用户设置的down-after-milliseconds选项的值，不仅会被Sentinel用来判断主服务器的主观下线状态，还会被用于判断主服务器属下的所有从服务器，以及所有同样监视这个主服务器的其他Sentinel 的主观下线状态。多个sentinel设置的主观下线时间可能不一样。

# 检查客观下线状态

当Sentinel将一个主服务器判断为主观下线之后，为了确认这个主服务器是否真的下线了，它会向同样监视这一主服务器的其他Sentinel进行询问，看它们是否也认为主服务器已经进入了下线状态（可以是主观下线或者客观下线)。当Sentinel 从其他Sentinel那里接收到足够数量的已下线判断之后，Sentinel就会将从服务器判定为客观下线，并对主服务器执行故障转移操作。

## 发送SENTINEL is-master-down-by-addr命令

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d059a3c600524bfe9a954b56bad1ef02~tplv-k3u1fbpfcp-zoom-1.image)

## 接收SENTINEL is-master-down-by-addr命令

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48d8fa0c767f4833a5f79e02f917582f~tplv-k3u1fbpfcp-zoom-1.image)

注意：runid/leader_runid为*是检测主观下线时期用的，客观下线后选举leader时不会为*


## 接收SENTINEL is-master-down-by-addr命令的回复

根据其他Sentinel发回的SENTINEL is-master-down-by-addr命令回复，Sentinel将统计其他Sentinel同意主服务器已下线的数量，当这一数量达到配置指定的判断客观下线所需的数量时，Sentinel会将主服务器实例结构flags属性的SRI_O_DOWN标识打开，表示主服务器已经进入客观下线状态，如图16-19所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8f6661a549f4f929daa0f72f0075319~tplv-k3u1fbpfcp-zoom-1.image)

**注意：不同Sentinel判断客观下线的条件可能不同**

# 选举领头Sentinel

以下是Redis选举领头 Sentinel的规则和方法:

-   -   所有在线的Sentinel都有被选为领头 Sentinel的资格，换句话说，监视同一个主服务器的多个在线Sentinel 中的任意一个都有可能成为领头Sentinel。
    -   每次进行领头Sentinel选举之后，不论选举是否成功，所有Sentinel的配置纪元(configuration epoch)的值都会自增一次。配置纪元实际上就是一个计数器，并没有什么特别的。

<!---->

-   -   在一个配置纪元里面，所有Sentinel都有一次将某个Sentinel设置为局部领头Sentinel的机会，并且局部领头一旦设置，在这个配置纪元里面就不能再更改。每个发现主服务器进入客观下线的Sentinel都会要求其他Sentinel将自己设置为局部领头Sentinel。
    -   当一个 Sentinel（源Sentinel)向另一个Sentinel(目标Sentinel）发送SENTINELis-master-down-by-addr命令，并且命令中的runid参数不是*符号而是源Sentinel的运行ID时，这表示源Sentinel要求目标Sentinel将前者设置为后者的局部领头Sentinel。

<!---->

-   -   Sentinel设置局部领头Sentinel的规则是先到先得：最先向目标Sentinel发送设置要求的源Sentinel将成为目标Sentinel的局部领头Sentinel，而之后接收到的所有设置要求都会被目标Sentinel拒绝。
    -   目标Sentinel在接收到SENTINEL is-master-down-by-addr命令之后，将向源Sentinel返回一条命令回复，回复中的 leader_runid参数和leader_epoch参数分别记录了目标Sentinel的局部领头 Sentinel的运行ID和配置纪元。

<!---->

-   -   源Sentinel在接收到目标Sentinel返回的命令回复之后，会检查回复中leader_epoch参数的值和自己的配置纪元是否相同，如果相同的话，那么源Sentinel继续取出回复中的leader_runid参数，如果leader_runid参数的值和源Sentinel的运行ID一致，那么表示目标Sentinel将源Sentinel设置成了局部领头 Sentinel
    -   如果有某个Sentinel被半数以上的Sentinel设置成了局部领头Sentinel，那么这个Sentinel成为领头 Sentinel。举个例子，在一个由10个Sentinel组成的Sentinel系统里面，只要有大于等于10/2+1=6个Sentinel将某个Sentinel 设置为局部领头Sentinel，那么被设置的那个Sentinel 就会成为领头 Sentinel。

<!---->

-   -   因为领头Sentinel的产生需要半数以上Sentinel的支持，并且每个Sentinel在每个配置纪元里面只能设置一次局部领头 Sentinel，所以在一个配置纪元里面，只会出现一个领头Sentinel。
    -   如果在给定时限内，没有一个Sentinel被选举为领头Sentinel，那么各个 Sentinel将在一段时间之后再次进行选举，直到选出领头Sentinel为止。

举个例子：

假设现在有三个 Sentinel正在监视同一个主服务器，并且这三个Sentinel之前已经通过SENTINEL is-master-down-by-addr命令确认主服务器进入了客观下线状态

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa2cb80e123d4266bd70e0e7d38a922a~tplv-k3u1fbpfcp-zoom-1.image)

那么为了选出领头Sentinel,三个Sentinel将再次向其他Sentinel 发送SENTINEL is-master-down-by-addr命令,如图16-21所示。

和检测客观下线状态时发送的SENTINEL is-master-down-by-addr命令不同,Sentinel这次发送的命令会带有Sentinel自己的运行ID，例如:

SENTINEL is-master-down-by-addr 127.0.0. 1 6379 o e955b4c85598ef5b5f055bc7ebfd5e828dbed4fa

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23c176ab42fd48edac1f89bb628adfc0~tplv-k3u1fbpfcp-zoom-1.image)

如果接收到这个命令的Sentinel还没有设置局部领头Sentinel 的话，它就会将运行ID为`e955b4c85598ef5b5f055bc7ebfd5e828dbed4fa`的Sentinel设置为自己的局部领头Sentinel，并返回类似以下的命令回复:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50114e96994d43c5a6e99e2ab99008c7~tplv-k3u1fbpfcp-zoom-1.image)

然后接收到命令回复的Sentinel就可以根据这一回复，统计出有多少个Sentinel将自己设置成了局部领头Sentinel。

根据命令请求发送的先后顺序不同，可能会有某个Sentinel的 SENTINEL is-master-down-by-addr命令比起其他Sentinel 发送的相同命令都更快到达，并最终胜出领头 Sentinel的选举，然后这个领头 Sentinel就可以开始对主服务器执行故障转移操作了。

# 故障转移

在选举产生出领头Sentinel之后，领头Sentinel将对已下线的主服务器执行故障转移操作，该操作包含以下三个步骤:

1.  在已下线主服务器属下的所有从服务器里面，挑选出一个从服务器，并将其转换为主服务器。
1.  让已下线主服务器属下的所有从服务器改为复制新的主服务器。

<!---->

3.  将已下线主服务器设置为新的主服务器的从服务器，当这个旧的主服务器重新上线时，它就会成为新的主服务器的从服务器。

## 选出新的主服务器

故障转移操作第一步要做的就是在已下线主服务器属下的所有从服务器中，挑选出一个状态良好、数据完整的从服务器，然后向这个从服务器发送SLAVEOF no one命令，将这个从服务器转换为主服务器。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c8549a23aa14f13924915e4204d693c~tplv-k3u1fbpfcp-zoom-1.image)

## 修改从服务器的复制目标

当新的主服务器出现之后，领头 Sentinel下一步要做的就是，让已下线主服务器属下的所有从服务器去复制新的主服务器，这一动作可以通过向从服务器发送SLAVEOF命令来实现。

图16-24展示了在故障转移操作中，领头Sentinel向已下线主服务器server1的两个从服务器server3和server4发送SLAVEOF命令，让它们复制新的主服务器server2的例子。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f184c1123ee24e4fb9e501d90b76e7c4~tplv-k3u1fbpfcp-zoom-1.image)

## 将旧的主服务器变为从服务器

故障转移操作最后要做的是，将已下线的主服务器设置为新的主服务器的从服务器。比如说，图16-26就展示了被领头 Sentinel设置为从服务器之后，服务器server1的样子。

因为旧的主服务器已经下线，所以这种设置是保存在server1对应的实例结构里面的，当server1重新上线时，Sentinel就会向它发送SLAVEOF命令，让它成为server2的从服务器。

# 总结

-   Sentinel 只是一个运行在特殊模式下的Redis服务器，它使用了和普通模式不同的命令表，所以Sentinel模式能够使用的命令和普通Redis服务器能够使用的命令不同。
-   Sentinel会读入用户指定的配置文件，为每个要被监视的主服务器创建相应的实例结构，并创建连向主服务器的命令连接和订阅连接，其中命令连接用于向主服务器发送命令请求，而订阅连接则用于接收指定频道的消息。

<!---->

-   Sentinel通过向主服务器发送 INFO命令来获得主服务器属下所有从服务器的地址信息，并为这些从服务器创建相应的实例结构，以及连向这些从服务器的命令连接和订阅连接。
-   在一般情况下，Sentinel 以每十秒一次的频率向被监视的主服务器和从服务器发送INFO命令，当主服务器处于下线状态，或者Sentinel正在对主服务器进行故障转移操作时，Sentinel向从服务器发送 INFO命令的频率会改为每秒一次。

<!---->

-   对于监视同一个主服务器和从服务器的多个Sentinel来说，它们会以每两秒一次的频率，通过向被监视服务器的_sentinel__:hello频道发送消息来向其他Sentinel宣告自己的存在。
-   每个 Sentinel也会从__sentinel_:hello频道中接收其他Sentinel 发来的信息，并根据这些信息为其他Sentinel创建相应的实例结构，以及命令连接。

<!---->

-   Sentinel 只会与主服务器和从服务器创建命令连接和订阅连接，Sentinel 与Sentinel之间则只创建命令连接。
-   Sentinel 以每秒一次的频率向实例（包括主服务器、从服务器、其他Sentinel )发送PING命令，并根据实例对PING命令的回复来判断实例是否在线，当一个实例在指定的时长中连续向Sentinel发送无效回复时，Sentinel会将这个实例判断为主观下线。当Sentinel将一个主服务器判断为主观下线时，它会向同样监视这个主服务器的其他Sentinel进行询问，看它们是否同意这个主服务器已经进入主观下线状态。

<!---->

-   当Sentinel收集到足够多的主观下线投票之后，它会将主服务器判断为客观下线，并发起一次针对主服务器的故障转移操作。


