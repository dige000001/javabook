## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 简介

## 持久化保存什么

1.  将当前数据状态进行保存，快照形式，存储数据结果，存储格式简单，关注点在数据
1.  将数据的操作过程进行保存，日志形式，存储操作过程，存储格式复杂，关注点在数据的操作过程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b81e12068aae4c0ea527bf8ea70d11bf~tplv-k3u1fbpfcp-zoom-1.image)

# RDB

## save指令

### 使用方式

save

将当前数据快照保存到配置文件里“logfile”指定的位置，格式为.rdb

注意：save指令的执行会阻塞当前redis服务器，直到rdb过程完成，线上环境不建议使用

### save相关配置（在conf中改）


**dbfilename dump.rdb**

说明：设置本地数据库文件名，默认值为dump.rdb

经验：通常设置为dump-端口号.rdb

**dir**

说明:设置存储.rdb文件的路径

经验:通常设置成存储空间较大的目录中，目录名称data

**rdbcompression yes**

说明:设置存储至本地数据库时是否压缩数据，默认为yes，采用LZF压缩

经验:通常默认为开启状态，如果设置为no，可以节省CPU运行时间，但会使存储的文件变大(巨大)

**rdbchecksum yes**

说明:设置是否进行RDB文件格式校验，该校验过程在写文件和读文件过程均进行

经验:通常默认为开启状态，如果设置为no，可以节约读写性过程约10%时间消耗，但是存储一定的数

据损坏风险

### 恢复

重新启动就自动加载恢复了

## bgsave

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8479883078a6454d958993c10198a78d~tplv-k3u1fbpfcp-zoom-1.image)

### 使用

bgsave

手动启动后台保存操作，不是立即执行

### 工作原理![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f045e57f119a411b8aa9a2e1655a45d0~tplv-k3u1fbpfcp-zoom-1.image)

### 相关配置

save的配置在这里也生效

**stop-writes-on-bgsave-erroryes**

说明:后台存储过程中如果出现错误现象，是否停止保存操作

经验:通常默认为开启状态

## 自动执行保存

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22f49c357a4a4e2d9541e75ff917b480~tplv-k3u1fbpfcp-zoom-1.image)

通过bgsave实现

### 配置文件

**save second changes**

作用：满足限定时间范围内key的变化数量达到指定数量即进行持久化参数

second：监控时间范围

changes：监控key的变化量

这些属性都被保存在redisServer中，此外，还有dirty计数器和lastsave属性，两者分别记录了上次执行save或bgsave后所有数据库进行了多少次修改和至今经过了多长时间。然后服务器的周期性函数serverCron默认每100ms执行一次，检查dirty和lastsave是否达到了自动保存的条件，是则执行bgsave

### 原理

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd767951e4354879811c7509c76d1f7e~tplv-k3u1fbpfcp-zoom-1.image)

注意：

save配置要根据实际业务情况进行设置，频度过高或过低都会出现性能问题，结果可能是灾难性的

save配置中对于second与changes设置通常具有互补对应关系，尽量不要设置成包含性关系

save配置启动后执行的是bqsave操作

## save和bgsave对比

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98b61e2c73a440328a891dacb9a058ca~tplv-k3u1fbpfcp-zoom-1.image)

注意：在bgsave命令执行期间，客户端发送的save和bgsave命令会被拒绝，bgrewriteof会被推迟到bgsave完成，而若在bgrewriteof执行期间调用bgsave，bgsave会被拒绝。

## rdb其他启动方式

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab6ac698d6a74c0287cb2797336a7636~tplv-k3u1fbpfcp-zoom-1.image)



## rdb优缺点

### RDB优点

-   RDB是一个紧凑压缩的二进制文件，存储效率较高
-   RDB内部存储的是redis在某个时间点的数据快照，非常适合用于数据备份，全量复制等场景RDB恢复数据的速度要比AOF快很多

<!---->

-   应用：服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复。

### RDB缺点

-   RDB方式无论是执行指令还是利用配置，无法做到实时持久化，具有较大的可能性丢失数据
-   bgsave指令每次运行要执行fork操作创建子进程，要牺牲掉一些性能

<!---->

-   Redis的众多版本中未进行RDB文件格式的版本统一，有可能出现各版本服务之间数据格式无法兼容现象

# AOF

### RDB存储的弊端

-   存储数据量较大，效率较低
-   基于快照思想，每次读写都是全部数据，当数据量巨大时，效率非常低大数据量下的IO性能较低

<!---->

-   基于fork创建子进程，内存产生额外消耗宕机带来的数据丢失风险

### 解决思路

-   不写全数据，仅记录部分数据
-   改记录数据为记录操作过程

<!---->

-   对所有操作均进行记录，排除丢失数据的风险

## AOF的概念

**AOF(append only file)持久化：** 以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中命令达到恢复数据的目的。与RDB相比可以简单描述为改记录数据为记录数据产生的过程

AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式

### AOF写数据的过程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6a716a23afb488bb74e1850038a7768~tplv-k3u1fbpfcp-zoom-1.image)

### AOF功能开启

appendonly yes| no

作用：是否开启AOF持久化功能，默认为不开启状态配置

appendfsync always | everysec | no

作用：AOF写数据策略

#### AOF三种写数据策略



-   **always(每次)**

每次写入操作均同步到AOF文件中，数据零误差，性能较低，不建议使用。

-   **everysec(每秒)**

每秒将缓冲区中的指令同步到AOF文件中，数据准确性较高，性能较高，建议使用，也是默认配置在系统突然宕机的情况下丢失1秒内的数据

-   **no(系统控制)**

由操作系统控制每次同步到AOF文件的周期，整体过程不可控

### AOF相关配置

appendfilename filename

作用：AOF持久化文件名，默认文件名为appendonly.aof，建议配置为`appendonly-端口号.aof`

dir

作用：AOF持久化文件保存路径，与RDB持久化文件保持一致即可

### AOF重写

随着命令不断写入AOF，文件会越来越大，为了解决这个问题，Redis引入了AOF重写机制压缩文件体积。AOF文件重写是将Redis进程内的数据转化为写命令同步到新AOF文件的过程。简单说就是将对同一个数据的若干个条命令执行结果转化成最终结果数据对应的指令进行记录。

-   降低磁盘占用量，提高磁盘利用率
-   提高持久化效率，降低持久化写时间，提高IO性能

<!---->

-   降低数据恢复用时，提高数据恢复效率

#### AOF重写方式

##### 手动重写

bgrewriteaof

##### 自动重写

auto-aof-rewrite-min-size size

auto-aof-rewrite-percentage percentage

##### 自动重写触发条件

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc49efbf7be84a8a8d72d13ea257d74d~tplv-k3u1fbpfcp-zoom-1.image)

#### 自动重写工作原理

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1607a4b4041a4468b93d81ab9b0ca4fa~tplv-k3u1fbpfcp-zoom-1.image)

#### AOF重写流程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c685449ff2134623a536a9a30acb6280~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70dfabf87ce9470884e95cc2628c8712~tplv-k3u1fbpfcp-zoom-1.image)

## AOF VS RDB

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41511aed299f409b8024ef3815bb52d0~tplv-k3u1fbpfcp-zoom-1.image)

### 如何选择



-   **对数据非常敏感，建议使用默认的AOF持久化方案**

<!---->

-   -   AOF持久化策略使用everysecond，每秒钟fsync一次。该策略redis仍可以保持很好的处理性能，当出现问题时，最多丢失0-1秒内的数据。
    -   注意:由于AOF文件存储体积较大，且恢复速度较慢

<!---->

-   **数据呈现阶段有效性，建议使用RDB持久化方案**

<!---->

-   -   数据可以良好的做到阶段内无丢失(该阶段是开发者或运维人员手工维护的)，且恢复速度较快，阶段点数据恢复通常采用RDB方案
    -   注意:利用RDB实现紧凑的数据持久化会使Redis降的很低

●**综合比对**

-   RDB与AOF的选择实际上是在做一种权衡，每种都有利有弊
-   如不能承受数分钟以内的数据丢失，对业务数据非常敏感，选用AOF

<!---->

-   如能承受数分钟以内的数据丢失，且追求大数据集的恢复速度，选用RDB
-   灾难恢复选用RDB

<!---->

-   双保险策略，同时开启RDB和AOF，重启后，Redis优先使用AOF来恢复数据，降低丢失数据的量


