## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1f6cc68debf438b952cfbd0c4b3093c~tplv-k3u1fbpfcp-zoom-1.image)



# 命令行工具



## jps：虚拟机进程状况工具 



jps(JVM Process Status Tool) :主要用来输出JVM中运行的进程状态信息，语法格式如下:

jps [ options ] [ hostid ]

hostid字符串的语法与URI的语法基本一致∶

[ protocol: ] [ //hostname ] [ :port ] [/servername] 

如果不指定hostid，默认为当前主机或服务器。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97700395c39a47299187c7404efbecc2~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a27fc14e30104264a7e23e41a38cdc79~tplv-k3u1fbpfcp-zoom-1.image)



## jinfo：Java配置信息工具 

jinfo（Configuration Info for Java）的作用是实时查看和调整虚拟机各项参数。使用jps命令的-v参 数可以查看虚拟机启动时显式指定的参数列表，但如果想知道未被显式指定的参数的系统默认值，除 了去找资料外，就只能使用jinfo的-flag选项进行查询了 

执行样例：查询CMSInitiatingOccupancyFraction参数值 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1080790596be4dff92b5082d597ea66e~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7898e42acbf642f4bc253e281edf119a~tplv-k3u1fbpfcp-zoom-1.image)

p.s:Windows环境下这个命令不太好用

## jstack：Java堆栈跟踪工具 

jstack（Stack Trace for Java）命令用于生成虚拟机当前时刻的线程快照（一般称为threaddump或者 javacore文件）。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的 目的通常是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间挂 起等，都是导致线程长时间停顿的常见原因。线程出现停顿时通过jstack来查看各个线程的调用堆栈， 就可以获知没有响应的线程到底在后台做些什么事情，或者等待着什么资源。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a529a8c892549f99ef45acab26dd8f9~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f23230491312401ea09089bc0ea2a5bd~tplv-k3u1fbpfcp-zoom-1.image)



## jmap：Java内存映像工具 

jmap（Memory Map for Java）命令用于生成堆转储快照（一般称为heapdump或dump文件）。如 果不使用jmap命令，要想获取Java堆转储快照也还有一些比较“暴力”的手段：譬如在第2章中用过的XX：+HeapDumpOnOutOfMemoryError参数，可以让虚拟机在内存溢出异常出现之后自动生成堆转储 快照文件，通过-XX：+HeapDumpOnCtrlBreak参数则可以使用[Ctrl]+[Break]键让虚拟机生成堆转储快 照文件，又或者在Linux系统下通过Kill-3命令发送进程退出信号“恐吓”一下虚拟机，也能顺利拿到堆转 储快照。 

jmap的作用并不仅仅是为了获取堆转储快照，它还可以查询finalize执行队列、Java堆和方法区的 详细信息，如空间使用率、当前用的是哪种收集器等。 

和jinfo命令一样，jmap有部分功能在Windows平台下是受限的，除了生成堆转储快照的-dump选项 和用于查看每个类的实例、空间占用统计的-histo选项在所有操作系统中都可以使用之外，其余选项都 只能在Linux/Solaris中使用。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0df432fa3fc43dcbe913774420ea7fb~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89be8d63d25a48098c244d2d255005e1~tplv-k3u1fbpfcp-zoom-1.image)

##

##

## jhat内存映像分析工具

和上面的jmap搭配使用，但是一般不用，有更好的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e082fa1d19044d8391269b20961be6ce~tplv-k3u1fbpfcp-zoom-1.image)

```
C:\Users\Hasee>jhat eclipse.bin
Reading from eclipse.bin...
Dump file created Tue Aug 17 18:23:12 CST 2021
Snapshot read, resolving...
Resolving 8393917 objects...
Chasing references, expect 1678 dots...
Eliminating duplicate references..
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

之后访问localhost:7000就能看到内存情况了

## jstat：虚拟机统计信息监视工具 

jstat（JVM Statistics Monitoring Tool）是用于监视虚拟机各种运行状态信息的命令行工具。它可 以显示本地或者远程虚拟机进程中的类加载、内存、垃圾收集、即时编译等运行时数据，在没有 GUI图形界面、只提供了纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的常用工具。 

jstat命令格式为：

jstat [ option vmid [interval[s|ms] [count]] ] 

对于命令格式中的VMID与LVMID需要特别说明一下：

-   -   如果是本地虚拟机进程，VMID与LVMID 是一致的；
    -   如果是远程虚拟机进程，那VMID的格式应当是：

[protocol:][//]lvmid[@hostname[:port]/servername] 

参数interval和count代表查询间隔和次数，如果省略这2个参数，说明只查询一次。假设需要每250 毫秒查询一次进程2764垃圾收集状况，一共查询20次，那命令应当是：

jstat -gc 2764 250 20 

选项option代表用户希望查询的虚拟机信息，主要分为三类：类加载、垃圾收集、运行期编译状 况。详细请参考表4-2中的描述

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0af41ed48774682aeb09e259be84050~tplv-k3u1fbpfcp-zoom-1.image)

jstat执行样例 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b21cd2bb2610438494b3be6657ed32f7~tplv-k3u1fbpfcp-zoom-1.image)

查询结果表明：这台服务器的新生代Eden区（E，表示Eden）使用了18.78%的空间，2个Survivor区 （S0、S1，表示Survivor0、Survivor1）分别使用了0%和100%，老年代（O，表示Old）和元数据区（M，表示 Meta）则分别使用了78.76%和95.47%的空间。程序运行以来共发生Minor GC（YGC，表示Young GC）195次，总耗时1.019秒；发生Full GC（FGC，表示Full GC）0次，总耗时（FGCT，表示Full GC Time）为0秒；所有GC总耗时（GCT，表示GC Time）为1.019秒。 

## jstated

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e05757864e24343be9b76ccb1d8bbad~tplv-k3u1fbpfcp-zoom-1.image)




## jcmd

几乎涵盖了上面所有命令的功能

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b755bbcd54ac4d04a519b3a0bf01d986~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a085b32e19447cea9a2296f97312729~tplv-k3u1fbpfcp-zoom-1.image)


其他工具参考《深入理解java虚拟机》P208



# 可视化工具

## jconsole

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46e98f4d34cc4ef194a41d4a46fb7b3b~tplv-k3u1fbpfcp-zoom-1.image)

在命令行里输入jconsole就可以打开

参考《深入理解java虚拟机》P221

## JHSDB

参考《深入理解java虚拟机》P214

## JMC (Java Mission Control：可持续在线的监控工具 )

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52e6f42701e64b05bcbf3446cdf96e49~tplv-k3u1fbpfcp-zoom-1.image)

## VisualVM 多合一故障处理工具 

VisualVM（All-in-One Java Troubleshooting Tool）是功能最强大的运行监视和故障处理程序之一， 曾经在很长一段时间内是Oracle官方主力发展的虚拟机故障处理工具。Oracle曾在VisualVM的软件说明 中写上了“All-in-One”的字样，预示着它除了常规的运行监视、故障处理外，还将提供其他方面的能 力，譬如性能分析（Profiling）。VisualVM的性能分析功能比起JProfiler、YourKit等专业且收费的 Profiling工具都不遑多让。而且相比这些第三方工具，VisualVM还有一个很大的优点：不需要被监视的 程序基于特殊Agent去运行，因此它的通用性很强，对应用程序实际性能的影响也较小，使得它可以直 接应用在生产环境中。这个优点是JProfiler、YourKit等工具无法与之媲美的。 

参考《深入理解java虚拟机》P229

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd375671b576478a8c10edddf7c73b9f~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1417536cba63421b9f4312b9b1b03df6~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6735cf82230942e6a709ef693bd268d3~tplv-k3u1fbpfcp-zoom-1.image)

