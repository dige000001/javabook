## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 一 CPU

在 Linux 中，可通过top或pidstat方式来查看进程中线程的CPU的消耗状况。

## 基础命令

### 1. top

输入 top命令后即可查看CPU 的消耗情况,CPU 的信息在TOP视图的上面几行中

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1d4e962ad7c4f35956dd6ba9911bb61~tplv-k3u1fbpfcp-zoom-1.image)

在此需要关注的是第三行的信息，其中us 表示为用户进程处理所占的百分比；sy表示为内核线程处理所占的百分比； ni表示被nice命令改变优先级的任务所占的百分比；id表示CPU的空闲时间所占的百分比；wa表示为在执行的过程中等待IO所占的百分比； hi表示为硬件中断所占的百分比；si表示为软件中断所占的百分比。

在这个页面按下shift+h，会切换到以线程为单位的CPU消耗情况，如下图

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93c098e5bb614ddfb4acac696aada99b~tplv-k3u1fbpfcp-zoom-1.image)

### 2.pidstat

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5f4ef667c514e75bbb572c4c5757d49~tplv-k3u1fbpfcp-zoom-1.image)

可以使用 pidstat -p [pid] -t 1来显示某个进程的线程情况

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4688f36f089b44efa0dfed2a2431f265~tplv-k3u1fbpfcp-zoom-1.image)


当CPU消耗严重时，主要体现在us、sy、wa或hi的值变高，wa的值是IO等待造成的，这个在之后的章节中阐述; hi的值变高主要为硬件中断造成的，例如网卡接收数据频繁的状况。

对于Java应用而言，CPU消耗严重主要体现在us、sy两个值上，来分别看看Java应用在这两个值高的情况下应如何寻找对应造成瓶颈的代码。

## us

当us值过高时，表示运行的应用消耗了大部分的CPU。在这种情况下，对于Java应用而言，最重要的为找到具体消耗CPU的线程所执行的代码，可采用如下方法做到。

首先通过linux提供的命令找到消耗CPU严重的线程及其ID，将此线程ID转化为十六进制的值。之后通过

**kill -3 [javapid]** 或**jstack** 的方式dump出应用的java线程信息，通过之前转化出的十六进制的值找到对应的nid值的线程。该线程即为消耗 CPU 的线程，在采样时须多执行几次上述的过程，以确保找到真实的消耗CPU的线程。

Java应用造成us高的原因主要是线程一直处于可运行(Runnable)状态，通常是这些线程在执行无阻塞、循环、正则或纯粹的计算等动作造成;另外一个可能也会造成us高的原因是频繁的GC。如每次请求都需要分配较多内存，当访问量高的时候就将导致不断地进行GC，系统响应速度下降，进而造成堆积的请求更多，消耗的内存更严重，最严重的时候有可能会导致系统不断进行Full GC，对于频繁GC的状况要通过分析JVM内存的消耗来查找原因。

对于执行线程无任何挂起动作，且一直执行，导致CPU没有机会去调度执行其他的线程，造成线程饿死的现象。对于这种情况，常见的一种优化方法是对这种线程的动作增加 Thread.sleep，以释放CPU的执行权，降低CPU的消耗。

例子：《分布式java应用》P196 RealP178

P228

## sy

当sy值高时，表示 Linux花费了更多的时间在进行线程切换，Java应用造成这种现象的主要原因是启动的线程比较多，且这些线程多数都处于不断的阻塞（例如锁等待、IO等待状态）和执行状态的变化过程中，这就导致了操作系统要不断地切换执行的线程，产生大量的上下文切换。在这种状况下，对Java应用而言，最重要的是找出线程要不断切换状态的原因，可采用的方法为通过**kill -3 [javapid**]或**jstack -l [javapid]** 的方式dump 出 Java应用程序的线程信息，查看线程的状态信息以及锁信息，找出等待状态或锁竞争过多的线程。

如果线程过多，可以适当减少线程数，如果线程数实在无法减少，可以使用协程：

在目前的Sun JDK实现中，创建并启动一个Thread对象就意味着运行了一个原生线程，当这个线程中有任何的阻塞动作（例如同步文件 IO、同步网络IO、锁等待、Thread.sleep等）时，这个线程就会被挂起，但仍然占据着线程的资源。当线程中的阻塞动作完成时，由操作系统来恢复线程的上下文，并调度执行，这是一种标准的遵循目前操作系统的实现方式，这种方式对于Java应用而言，当并发量上涨后，有可能出现的现象是启动的大量线程都处于浪费状态。例如一个线程在等待数据库执行结果的返回，如这个数据库执行操作需要花费⒉秒，那么就意味着这个线程资源被白白占用了2秒，一方面导致了其他的请求只能是放在缓冲队列中等待执行，性能下降;另一方面是造成系统中线程切换频繁，CPU运行队列过长，协程要改变的就是不浪费相对昂贵的原生线程资源。

采用协程后，能做到当线程等待数据库执行结果时，就立即释放此线程资源给其他请求，等到数据库执行结果返回后才继续执行，在Java中目前主要可用于实现协程的框架为Kilim。在**使用Kilim执行一项任务时，并不创建Thread，而是改为创建Task，Task相对于Thread而言就轻量级多了。当此Task要做阻塞动作时，可通过Mailbox.get或Task.pause来阻塞当前Task，Kilim会保存Task之后执行需要的对象信息，并释放Task执行所占用的线程资源:当Task的阻塞动作完成或被唤醒时，此时Kilim会重新载入Task所需的对象信息，恢复Task的执行，相当于Kilim来承担了线程的调度以及上下文切换动作。** 这种方式相对原生Thread方式更为轻量，且能够更好地利用CPU，因此可做到仅启动CPU核数的线程数，以及大量的Task来支撑高并发量,Kilim带来的是线程使用率的提升，但同时由于要在JVM堆中保存Task上下文信息，因此在采用Kilim的情况下要消耗更多的内存。

例子：《分布式java应用》P198 P223

# 二 文件IO消耗

输入如 pidstat -d -t -p [pid] 1100类似的命令即可查看线程的IO消耗状况

当文件IO消耗过高时，对于Java应用最重要的是找到造成文件IO消耗高的代码，寻找的最佳方法为通过pidstat直接找到文件IO操作多的线程。之后结合 jstack 找到对应的Java 代码,如没有pidstat,也可直接根据jstack得到的线程信息来分析其中文件IO操作较多的线程。

Java应用造成文件IO消耗严重主要是多个线程需要进行大量内容写入（例如频繁的日志写入)的动作;或磁盘设备本身的处理速度慢;或文件系统慢;或操作的文件本身已经很大造成的。

常用调优方式：

**异步写文件**

将写文件的同步动作改为异步动作，避免应用由于写文件慢而性能下降太多，例如写日志，可以使用 log4j提供的AsyncAppender。

**批量读写**

频繁的读写操作对IO消耗会很严重，批量操作将大幅度提升IO操作的性能。

**限流**

频繁读写的另外一个调优方式是限流，从而将文件IO消耗控制到一个能接受的范围

**限制文件大小**

操作太大的文件也是造成文件IO效率低的一个原因，因此对于每个输出的文件，都应做大小的限制，在超出最大值后可生成一个新的文件，类似log4j 中 RollingFileAppender的maxFileSize 属性的作用。

除了以上这些外，还有就是尽可能采用缓冲区等方式来读取文件内容，避免不断与操作系统交互，

例子：《分布式java应用》P202 P225

# 三 网络IО消耗分析

sar -n 用不了啊……

由于没办法分析具体每个线程所消耗的网络IO，因此当网络IO消耗高时，对于Java应用而言只能对线程进行 dump，查找产生了大量网络IO操作的线程。这些线程的特征是读取或写入网络流，在用Java 实现网络通信时，通常要将对象序列化为字节流，进行发送，或读取字节流，并反序列化为对象。这个过程要消耗JVM堆内存，JVMJVM堆的内存大小通常是有限的，因此Java 应用一般不会造成网络IO消耗严重。

从程序角度而言，造成网络IO消耗严重的原因主要是同时需要发送或接收的包太多。对于这类情况，常用的调优方法为进行限流，限流通常是限制发送packet的频率，从而在网络IO消耗可接受的情况下来发送packet。

# 内存消耗分析

Java应用对于内存的消耗主要是在JVM堆内存上，在正式环境中，多**数Java应用都会将-Xms 和-Xmx设为相同的值，避免运行期要不断申请内存。**

在Linux中可通过vmstat、sar、top、pidstat等方式来查看swap和物理内存的消耗状况。

## vmstat

在命令行中输入 vmstat，其中的信息和内存相关的主要是 memory 下的 swpd、free、buff、cache以及swap 下的si和 so。

其中 swpd是指虚拟内存已使用的部分，单位为kb;free表示空闲的物理内存, buff表示用于缓冲的内存，cache表示用于作为缓存的内存，swap下的 si是指每秒从disk读至内存的数据量, so是指每秒从内存中写入 disk的数据量。

swpd值过高通常是由于物理内存不够用了，os将物理内存中的一部分数据转为放入硬盘上进行存储，以腾出足够的空间给当前运行的程序使用。在目前运行的程序变化后，即从硬盘上重新读取数据到内存中，以便恢复程序的运行，这个过程会产生swap IO，因此看swap的消耗情况主要要关注的是swap IO的状况，如 swap IO 发生得较频繁，那么会严重影响系统的性能。

由于Java应用是单进程应用，因此只要JVM的内存设置不是过大，是不会操作到swap区域的。物理内存消耗过高可能是由于JVM内存设置过大、创建的Java线程过多或通过Direct ByteBuffer往物理内存中放置了过多的对象造成的。

## sar

通过sar的-r参数可查看内存的消耗状况

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b280eeadcdb44549d6e31972dacf88e~tplv-k3u1fbpfcp-zoom-1.image)



```
public static void main (String[] args) throws Exception{
	Thread.sleep(20000) ;
	System.out.println ("read to create bytes,so JVM heap will be used" ) ;
    byte[] bytes=new byte[128*1000*1000];
	bytes [0]=1;
	bytes[1]=2 ;
	Thread.sleep(10000) ;
	System. out.println("read to allocate & put direct bytebuffer,no JVM heap should be used" ) ;
	ByteBuffer buffer=ByteBuffer.allocateDirect (128*1024*1024);
    buffer.put (bytes) ;
	buffer. flip();
	Thread .sleep(10000) ;
	System. out.println ( ready to gc, JVM heap will be freed" ) ;
    bytes=null;
	System.gc( );
	Thread.sleep(10000) ;
	System.out.println ("read to get bytes,then JVM heap will be used" ) ;
    byte[] resultbytes=new byte[128*1000*1000] ;
	buffer.get (resultbytes) ;
	System.out.println ( "resultbytes[1] is: "+resultbytes [1]);
    Thread.sleep (10000) ;
	System.out.println ( " read to gc all" );
    buffer=null;
	resultbytes=nu11;
	system.gc();
	Thread.sleep (10000);
                  }
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03697975243342de8732abae09705143~tplv-k3u1fbpfcp-zoom-1.image)

无变化是因为jvm里面的堆内存被回收了，但是没有释放，所以java进程还是占那么多内存

一般来说，对于内存调优，除了调整堆内存各区大小之外，还有以下技巧：

1.  **释放不必要的引用**
1.  **使用对象缓存池，但要注意缓存池大小，因为缓存池一直持有着对象的引用**

<!---->

3.  **采用合理的缓存失效算法**
3.  **合理使用软引用和弱引用**

例子：《分布式java应用》P227

# 程序执行慢原因分析

有些情况是资源消耗不多，但程序执行仍然慢，这种现象多出现于访问量不是非常大的情况下，造成这种现象的原因主要有以下三种:

**1．锁竞争激烈**

锁竞争激烈直接就会造成程序执行慢，例如一个典型的例子是数据库连接池，通常数据库连接池提供的连接数都是有限的。假设提供的是10个，那么就意味着同时能够进行数据库操作的就只有10个线程，而如果此时有50个线程要进行数据库操作，那就会造成另外的40个线程处于等待状态，这种情况下对于4核类型的机器而言，CPU的消耗并不会高，但程序的执行仍然会较慢。

**2．未充分使用硬件资源**

例如机器上有双核CPU，但程序中都是单线程串行的操作，并没有充分发挥硬件资源的作用，此时就可进行一定的优化来充分使用硬件资源，提升程序的执行速度。

**3．数据量增长**

数据量增长通常也是造成程序执行慢的典型原因，例如当数据库中单表的数据从100万个上涨到亿个后，数据库的读写速度将大幅度下降，相应的操作此表的程序的执行速度也就下降了。

对于以上这两种状况，要记录程序执行的整个过程的时间消耗或使用JProfiler等商业工具，从而找到执行耗时比率最大的代码

例子：《分布式java应用》P232

