## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 垃圾回收算法

1.  引用计数法

给对象添加一个引用计数器，有访问就加1，引用失效就减1

优点：实现简单、效率高;

缺点：不能解决对象之间循环引用的问题

2.  **根搜索算法**

从根（GC Roots )节点向下搜索对象节点，搜索走过的路经称为引用链，当一个对象到根之间没有连通的话，则该对象不可用

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/afcff596c3c34df4a460a610e6f4d9c2~tplv-k3u1fbpfcp-zoom-1.image)

可作为GC Roots的对象包括︰

```
·在虚拟机栈（栈帧中的本地变量表）中引用的对象，譬如各个线程被调用的方法堆栈中使用到的参数、局部变量、临时变量等。
·在方法区中类静态属性引用的对象，譬如Java类的引用类型静态变量。
·在方法区中常量引用的对象，譬如字符串常量池（String Table）里的引用。
·在本地方法栈中JNI（即通常所说的Native方法）引用的对象。
·Java虚拟机内部的引用，如基本数据类型对应的Class对象，一些常驻的异常对象（比如NullPointExcepiton、OutOfMemoryError）等，还有系统类加载器。
·所有被同步锁（synchronized关键字）持有的对象。
·反映Java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等。
```

注意： 是通过 **一系列称为“GC Roots”的根对象作为起始节点集**，从这些节点开始，根据引用关系向下搜索，搜索过 程所走过的路径称为“引用链”（Reference Chain），如果某个对象到GC Roots间没有任何引用链相连， 或者用图论的话来说就是从GC Roots到这个对象不可达时，则证明此对象是不可能再被使用的。 

HotSpot使用了一组叫做OopMap的数据结构达到准确式GC的目的

```
OopMap： 
	一旦类加载动作完成的时候， HotSpot就会把对象内什么偏移量上是什么类型的数据计算出来，
在即时编译过程中，也会在特定的位置记录下栈里和寄存器里哪些位置是引用。
这样收集器在扫描时就可以直接得知这些信息了，并不需要真正一个不漏地从方法区等GC Roots开始查找。  
```

在OopMap的协助下，JVM可以很快的做完GC Roots枚举。但是JVM并没有为每一条指令生成一个OopMap记录OopMap的这些“特定位置”被称为**安全点**，即所有线程执行到安全点后才允许暂停进行GC

#### 如何在垃圾收集发生时让所有线程（这里其实不包括 执行JNI调用的线程）都跑到最近的安全点 

主动式中断的思想是当垃圾收集需要中断线程的时候，不直接对线程操作，仅仅简单地设置一 个标志位，各个线程执行过程时会不停地主动去轮询这个标志，一旦发现中断标志为真时就自己在最 近的安全点上主动中断挂起。轮询标志的地方和安全点是重合的，另外还要加上所有创建对象和其他 需要在Java堆上分配内存的地方，这是为了检查是否即将要发生垃圾收集，避免没有足够内存分配新 对象。 

由于轮询操作在代码中会频繁出现，这要求它必须足够高效。HotSpot使用内存保护陷阱的方式， 把轮询操作精简至只有一条汇编指令的程度。下面代码清单3-4中的test指令就是HotSpot生成的轮询指 令，当需要暂停用户线程时，虚拟机把0x160100的内存页设置为不可读，那线程执行到test指令时就会 产生一个自陷异常信号，然后在预先注册的异常处理器中挂起线程实现等待，这样仅通过一条汇编指 令便完成安全点轮询和触发线程中断了。 

```
0x01b6d627: call 0x01b2b210 ; OopMap{[60]=Oop off=460}; *invokeinterface size
																											; - Client1::main@113 (line 23)
																											; {virtual_call}
0x01b6d62c: nop ; OopMap{[60]=Oop off=461}
																											; *if_icmplt
																											; - Client1::main@118 (line 23)
0x01b6d62d: test %eax,0x160100 ; {poll}
0x01b6d633: mov 0x50(%esp),%esi
0x01b6d637: cmp %eax,%esi
```

但如果用户线程处于Sleep状态或者Blocked状态，这时候线程无法响应虚拟机的中断请求，不能再走到安全的地方去中断挂起自己，虚拟机也显然不可能持续等待线程重新被激活分配处理器时间。对于 这种情况，就必须引入**安全区域**（Safe Region）来解决。 

如果一段代码中，对象引用关系不会发生变化，这个区域中任何地方开始GC都是安全的，那么这个区域称为安全区域

当用户线程执行到安全区域里面的代码时，首先会标识自己已经进入了安全区域，那样当这段时 间里虚拟机要发起垃圾收集时就不必去管这些已声明自己在安全区域内的线程了。当线程要离开安全 区域时，它要检查虚拟机是否已经完成了根节点枚举（或者垃圾收集过程中其他需要暂停用户线程的 阶段），如果完成了，那线程就当作没事发生过，继续执行；否则它就必须一直等待，直到收到可以 离开安全区域的信号为止 

# 引用分类

-   强引用：类似于Object a = new A()这样的，不会被回收
-   软引用：还有用但并不必须的对象。用SoftReference来实现软引用，当经过gc之后，内存还是不够，就可能回收软引用的对象空间

<!---->

-   弱引用：非必须对象，比软引用还要弱，垃圾回收时会回收掉。用WeakReference来实现弱引用
-   虚引用：也称为幽灵引用或幻影引用，是最弱的引用。垃圾回收时会回收掉。 为一个对象设置虚引用关联的唯一目的只是为了能在这个对象被收集器回收时收到一个系统通知。可以使用PhantomReference类来实现虚引用

注：一个对象可以同时有多种引用

**让我们来测试一下！：**

**user类**

```
public class User {
    private String userId;
    
	public User(String userId) {
		this.userId = userId;
	}
    
	public String toString( ) {
		return "UserId=="+userId;
	}
    
	@Override
	protected void finalize( ) throws Throwable {
		super.finalize();
		System.out.println( "now finalize userId=="+userId)
	}
}
```

## 弱引用

```
public class TestWeakRef {
    private static ReferenceQueue<User> rq = new ReferenceQueue<User>();

    private static void printQueue(String str) {
        //
        Reference<? extends User> obj = rq.poll();
        if (obj != null) {
            System.out.println("the gc 0jbect reference==" + str + " = " + obj);
        }
    }

    private static void testWeakReference() throws Exception {
        List<WeakReference<User>> list = new ArrayList<WeakReference<User>>();
        for (int i = 0; i < 10; i++) {
            String str = "soft" + i;
            WeakReference<User> sr = new WeakReference<User>(new User(str), rq);
            System.out.println("now the soft user===" + sr.get());
            list.add(sr);
            System.out.println("现在调用GC");
            System.gc();
            Thread.sleep(1000L);
            printQueue("soft");
        }
    }
    @Test
    public void main() throws Exception {
        testSoftReference();
        System.out.println(Runtime.getRuntime().maxMemory()/1024/1024);
        }
    }
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/660a0f2e894d43a285596172f195947d~tplv-k3u1fbpfcp-zoom-1.image)

可以看到，每次调用System.gc()都会回收弱引用的对象

## 软引用

```
public class TestSoftRef {
    private static ReferenceQueue<User> rq = new ReferenceQueue<User>();

    private static void printQueue(String str) {
        Reference<? extends User> obj = rq.poll();
        if (obj != null) {
            System.out.println("the gc 0jbect reference==" + str + " = " + obj);
        }
    }

    private static void testSoftReference() throws Exception {
        List<SoftReference<User>> list = new ArrayList<SoftReference<User>>();
        for (int i = 0; i < 5; i++) {
            String str = "soft" + i;
            SoftReference<User> sr = new SoftReference<User>(new User(str), rq);
            System.out.println("now the soft user===" + sr.get());
            list.add(sr);
            System.out.println("现在调用GC");
            System.gc();
            Thread.sleep(1000L);
            printQueue("soft");
        }
    }
    @Test
    public void main() throws Exception {
        testSoftReference();
        System.out.println(Runtime.getRuntime().maxMemory()/1024/1024);
        }
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a83d476c305497c890160dab37e64fd~tplv-k3u1fbpfcp-zoom-1.image)

可以看到，内存足够的情况下System.gc()不会回收软引用对象

那么，新增一下参数配置再run一次

-Xmx1m

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92a921cb1581472bae4b18a2e2c278f2~tplv-k3u1fbpfcp-zoom-1.image)

可以看到，除了第一次生成软引用还有内存所以不会清除之外，之后每次生成软引用且调用gc都会因为内存不足而清理掉软引用！

**代码说明：**

```
 User savePoint = new User("Random"); // 创建一个强引用
            ReferenceQueue<User> savepointQ = new ReferenceQueue<User>();//引用队列
 			//下面这代码的意思是如果该弱引用对象被回收了就将其加入savepointQ队列
            WeakReference<User> savePointWRefernce = new WeakReference<User>(savePoint, savepointQ);
            System.out.println("SavePoint 被作为一个弱引用来创建" + savePointWRefernce);
            Runtime.getRuntime().gc();
            System.out.println("在引用队列中存在引用型对象吗 ? " + (savepointQ.poll() != null));
            savePoint = null; // 唯一的强引用被删除掉，在堆中的对象现在只具有弱可达性
            System.out.println("现在调用GC");
            Runtime.getRuntime().gc(); // 对象会在这里被回收掉，finalize方法会被调用
            Thread.sleep(1000L);
            System.out.println("在引用队列中有任何弱引用吗? 若有，则remove" + (savepointQ.poll() != null));
            System.out.println("在引用队列中有任何弱引用吗? " + (savepointQ.poll() != null));
            System.out.println("弱引用型对象还引用着堆中的对象吗?" + (savePointWRefernce.get() != null));
            System.out.println("弱引用型对象被添加到引用队列中了吗?" + (savePointWRefernce.isEnqueued()));
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bad4245918124c93906a726f5339aa7f~tplv-k3u1fbpfcp-zoom-1.image)


# 垃圾回收基础

## 跨代引用

也就是一个代中的对象引用另一个代中的对象，跨代引用相对于同代引用来说只是极少数，存在互相引用关系的两个对象，是应该倾向于同时生存或同时消亡的

## 记忆集(Remembered Set )和卡表

一种用于记录从非收集区域指向收集区域的指针集合的抽象数据结构，简单地说就是记录了跨代引用的数据结构。 垃圾收集器在新生代中建立了名为记忆集（Remembered Set）的数据结构，用以避免把整个老年代加进GC Roots扫描范围 ，记录集一般有以下精度：

-   字长精度：每个记录精确到程序中的一个字，该字包含跨代指针。
-   对象精度：每个记录精确到程序中的一个对象，该对象里有字段含有跨代指针。

<!---->

-   **卡精度**：每个记录精确到程序中的一块内存区域，该区域内有对象含有跨代指针。现在一般用这个

**卡表(Card Table )**  :是记忆集的一种具体实现，定义了记忆集的记录精度和与堆内存的映射关系等；卡表的每个元素都对应着其标识的内存区域中一块特定大小的内存块，这个内存块称为卡页(Card Page ) 

一个卡页的内存中通常包含不止一个对象，只要卡页内有一个（或更多）对象的字段存在着跨代 指针，那就将对应卡表的数组元素的值标识为1，称为这个元素变脏（Dirty），没有则标识为0。在垃 圾收集发生时，只要筛选出卡表中变脏的元素，就能轻易得出哪些卡页内存块中包含跨代指针，把它 们加入GC Roots中一并扫描。 

### 写屏障

写屏障可以看成是JVM对“引用类型字段赋值”这个动作的AOP，通过写屏障来实现当对象状态改变后，维护卡表状态

### 伪共享

伪共享是处理并发底层细节时一种经常需要考虑的问题，现代中央处理器的缓存系统中是以缓存行（Cache Line） 为单位存储的，当多线程修改互相独立的变量时，如果这些变量恰好共享同一个缓存行，就会彼此影响（写回、无效化或者同步）而导致性能降低，这就是伪共享问题。 

假设处理器的缓存行大小为64字节，由于一个卡表元素占1个字节，64个卡表元素将共享同一个缓 存行。这64个卡表元素对应的卡页总的内存为32KB（64×512字节），也就是说如果不同线程更新的对 象正好处于这32KB的内存区域内，就会导致更新卡表时正好写入同一个缓存行而影响性能。为了避免 伪共享问题，一种简单的解决方案是不采用无条件的写屏障，而是先检查卡表标记，只有当该卡表元素未被标记过时才将其标记为变脏 

在JDK 7之后，HotSpot虚拟机增加了一个新的参数-XX： **+UseCondCardMark**，用来决定是否开启 卡表更新的条件判断。开启会增加一次额外判断的开销，但能够避免伪共享问题，两者各有性能损 耗，是否打开要根据应用实际运行情况来进行测试权衡 

###

## 判断是否为垃圾

1.  根搜索算法判断不可用
1.  看是否有必要执行finalize方法（注意：**一个对象的finalize方法只会执行一次**，就算你在第一次finalize里实现了自救，之后若这个对象再次没有引用了，gc就不会管你有没有finalize了，而是直接回收这个对象）

事实上，finalize方法没什么卵用，有些教材中描述它适合做“关闭外部资源”之类的清理性工作，这完全是对finalize() 方法用途的一种自我安慰。finalize()能做的所有工作，使用try-finally或者其他方式都可以做得更好、 更及时，所以建议完全忘掉Java语言里面的这个方法。 

3.  两个步骤走完后对象仍然没有人使用，那就属于垃圾



## GC类型

-   **MinorGC/YoungGC：** 发生在新生代的收集动作
-   **MajorGC / OldGC：** 发生在老年代的GC，目前只有CMS收集器会有单独收集老年代的行为

<!---->

-   **MixedGC：** 收集整个新生代以及部分老年代，目前只有G1收集器会有这种行为
-   **FullGC：** 收集整个Java堆和方法区的GC

除直接调用System.gc外，触发Full GC执行的情况有如下四种。

**1．旧生代空间不足**

旧生代空间只有在新生代对象转入及创建为大对象、大数组时才会出现不足的现象,当执行Full GC后空间仍然不足，则抛出如下错误: java.lang. OutOfMemoryError: Java heap space

**2．Permanet Generation空间满**

Permanet Generation中存放的为一些class的信息等，当系统中要加载的类、反射的类和调用的方法较多时，Permanet Generation可能会被占满，在未配置为采用CMSGC的情况下会执行Full GC。如果经过 Full GC仍然回收不了，那么JVM会抛出如下错误信息:java.lang. OutOfMemoryError: PermGen space

**3.CMS GC时出现promotion failed和concurrent mode failure**

对于采用CMS进行旧生代GC 的程序而言，尤其要注意GC日志中是否有 promotion failed 和concurrent mode failure两种状况，当这两种状况出现时可能会触发Full GC。

promotion failed是在进行Minor GC时，survivor space放不下、对象只能放入旧生代，而此时旧生代也放不下造成的; concurrent mode failure是在执行CMS GC的过程中同时有对象要放入旧生代，而此时旧生代空间不足造成的。

**4.统计得到的Minor GC晋升到旧生代的平均大小大于旧生代的剩余空间**

## Stop-The-World

GC回收的时候，需要用户线程暂时停止，防止对象之间的引用发生变化，不利于垃圾回收，这就是STW，在这个过程中，所有Java代码停止运行，native代码可以执行，但不能和JVM交互。

其危害是长时间服务停止，没有响应;对于HA系统，可能引起主备切换，严重危害生产环境

## 垃圾收集类型

**串行收集:** GC单线程内存回收、会暂停所有的用户线程，如: Serial

**并行收集:** 多个GC线程并发工作，此时用户线程是暂停的，如: Parallel

**并发收集∶**用户线程和GC线程同时执行（不一定是并行，可能交替执行），不需要停顿用户线程，如:CMS

## 方法区的回收

方法区的回收是卸载类，即将一个类的元数据从方法区中删除，方法区的回收有以下几个条件：

1.  JVM中该类的所有实例都已经被回收
1.  加载该类的ClassLoader已经被回收

<!---->

3.  没有任何地方引用该类的Class对象
3.  无法在任何地方通过反射访问这个类

方法区垃圾收集的“性价比”通常是比较低的：在Java堆中，尤其是在新生代中，对常规应用进行一次垃圾收集通常 可以回收70%至99%的内存空间，相比之下，方法区回收囿于苛刻的判定条件，其区域垃圾收集的回 收成果往往远低于此。 

# 垃圾回收算法：

## 标记清除法（Mark-Sweep ）

算法分成标记和清除两个阶段，先标记出要回收的对象，然后统一回收这些对象

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0320305bc3b46e2bd2580121e9a8181~tplv-k3u1fbpfcp-zoom-1.image)

优点是简单

缺点是∶

1.  效率不高，标记和清除的效率都不高
1.  标记清除后会产生大量不连续的内存碎片，从而导致在分配大对象时触发GC

## 复制算法(Copying)

把内存分成两块完全相同的区域，每次使用其中一块，当一块使用完了，就把这块上还存活的对象拷贝到另外一块，然后把这块清除掉。

优点是︰实现简单，运行高效，不用考虑内存碎片问题

缺点是∶内存有些浪费

JVM实际实现中，是将内存分为一块较大的Eden区和两块较小的Survivor空间，每次使用Eden和一块Survivor，回收时，把存活的对象复制到另一块Survivor.

HotSpot默认的Eden和Survivor比是8:1，也就是每次能用90%的新生代空间如果Survivor空间不够，就要依赖老年代进行**分配担保**，把放不下的对象直接进入老年代

**分配担保步骤：**

1.  在发生MinorGC前，JVM会检查老年代的最大可用的连续空间，是否大于新生代所有对象的总空间，如果大于，可以确保MinorGC是安全的
1.  如果小于，那么JVM会检查是否设置了允许担保失败，如果允许，则继续检查老年代最大可用的连续空间，是否大于历次晋升到老年代对象的平均大小，如果大于就进行一次MinorGC，否则，进行FullGC

## 标记整理算法(Mark-Compact )

由于复制算法在存活对象比较多的时候，效率较低，且有空间浪费，因此老年代一般不会选用复制算法，老年代多选用标记整理算法，标记过程跟标记清除一样，但后续不是直接清除可回收对象，而是让所有存活对象都向一端移动，然后直接清除边界以外的内存

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e69570f0c5ef4276997dc078d2d3e7a7~tplv-k3u1fbpfcp-zoom-1.image)




# 并发的可达性分析

我们引入三色标记（Tri-color Marking）作为工具来辅 助推导，把遍历对象图过程中遇到的对象，按照“是否访问过”这个条件标记成以下三种颜色： 

**·白色：** 表示对象尚未被垃圾收集器访问过。显然在可达性分析刚刚开始的阶段，所有的对象都是白色的，若在分析结束的阶段，仍然是白色的对象，即代表不可达。

**·黑色：** 表示对象已经被垃圾收集器访问过，且这个对象的所有引用（指这个对象对其他对象的引用）都已经扫描过。黑色的对象代表已经扫描过，它是安全存活的，如果有其他对象引用指向了黑色对象，无须重新扫描一遍。黑色对 象不可能直接（不经过灰色对象）指向某个白色对象。

**·灰色：** 表示对象已经被垃圾收集器访问过，但这个对象上至少存在一个引用（指这个对象对其他对象的引用）还没有被扫描过。 

关于可达性分析的扫描过程，读者不妨发挥一下想象力，把它看作对象图上一股以灰色为波峰的 波纹从黑向白推进的过程，如果用户线程此时是冻结的，只有收集器线程在工作，那不会有任何问 题。但如果用户线程与收集器是并发工作呢？收集器在对象图上标记颜色，同时用户线程在修改引用 关系——即修改对象图的结构，这样可能出现两种后果。一种是把原本消亡的对象错误标记为存活， 这不是好事，但其实是可以容忍的，只不过产生了一点逃过本次收集的浮动垃圾而已，下次收集清理 掉就好。另一种是把原本存活的对象错误标记为已消亡，这就是非常致命的后果了，程序肯定会因此 发生错误，下面表3-1演示了这样的致命错误具体是如何产生的。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecbeca7dd1064625a83233b981499990~tplv-k3u1fbpfcp-zoom-1.image)

Wilson于1994年在理论上证明了，当且仅当以下两个条件同时满足时，会产生“对象消失”的问 题，即原本应该是黑色的对象被误标为白色：

·赋值器插入了一条或多条从黑色对象到白色对象的新引用；

·赋值器删除了全部从灰色对象到该白色对象的直接或间接引用 

因此，我们要解决并发扫描时的对象消失问题，只需破坏这两个条件的任意一个即可。由此分别 产生了两种解决方案：增量更新（Incremental Update）和原始快照（Snapshot At The Beginning， SATB）。 

**增量更新**要破坏的是第一个条件，当黑色对象插入新的指向白色对象的引用关系时，就将这个新 插入的引用记录下来，等并发扫描结束之后，再将这些记录过的引用关系中的黑色对象为根，重新扫 描一次。这可以简化理解为，黑色对象一旦新插入了指向白色对象的引用之后，它就变回灰色对象 了。 

**原始快照**要破坏的是第二个条件，当灰色对象要删除指向白色对象的引用关系时，就将这个要删除的引用记录下来，在并发扫描结束之后，再将这些记录过的引用关系中的灰色对象为根，重新扫描 一次。这也可以简化理解为，无论引用关系删除与否，都会按照刚刚开始扫描那一刻的对象图快照来 进行搜索。 

以上无论是对引用关系记录的插入还是删除，虚拟机的记录操作都是通过写屏障实现的。在 HotSpot虚拟机中，增量更新和原始快照这两种解决方案都有实际应用，譬如，CMS是基于增量更新 来做并发标记的，G1、Shenandoah则是用原始快照来实现。 

# 垃圾收集器

前面讨论的垃圾收集算法只是内存回收的方法，垃圾收集器就来具体实现这些这些算法并实现内存回收

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe1d7e21942d43d8addcc6dd4377e86c~tplv-k3u1fbpfcp-zoom-1.image)

## 串行收集器

Serial （串行）收集器/Serial Old收集器，是一个单线程的收集器，在垃圾收集时，会Stop-the-World

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/135ccabad3b74ca0ab846d223f6fb479~tplv-k3u1fbpfcp-zoom-1.image)

-   优点是简单，对于单cpu，由于没有多线程的交互开销，可能更高效，是默认的Client模式下的新生代收集器
-   使用;XX:+UseSerialGC来开启，会使用:Serial + SerialOld的收集器组合

<!---->

-   新生代使用复制算法，老年代使用标记-整理算法

## 并行收集器

### ParNew(并行）收集器∶

使用多线程进行垃圾回收，在垃圾收集时，会Stop-the-World

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56028b13b5744bf092bbb643c755fb97~tplv-k3u1fbpfcp-zoom-1.image)




-   在并发能力好的CPU环境里，它停顿的时间要比串行收集器短﹔但对于单cpu或并发能力较弱的CPU，由于多线程的交互开销，可能比串行回收器更差
-   新生代使用复制算法

<!---->

-   是Server模式下首选的新生代收集器，且能和CMS收集器配合使用
-   开启：-XX:+UseconcMarkSweepGC -XX:ParallelGCThreads:指定线程数，最好与CPU数量一致


### Parallel Scavenge/Parallel Old收集器

Parallel Scavenge是一个应用于新生代的、使用复制算法的、并行的收集器；跟ParNew很类似，但更关注吞吐量，能最高效率的利用CPU，适合运行后台应用

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30d415bbec894c9982ca7f69dba2d898~tplv-k3u1fbpfcp-zoom-1.image)

-   新生代使用复制算法，老年代使用标记-整理算法
-   使用-XX:+UseParallelGC来开启

<!---->

-   使用-XX:+UseParallelOldGC来开启Parallel Scavenge + Parallel Old的收集器组合
-   -XX:MaxGCPauseMillis:设置GC的最大停顿时间




### CMS收集器( Concurrent Mark and Sweep并发标记清除）

CMS进行垃圾回收时，用户线程不会停顿

步骤∶

1.  初始标记∶只标记GC Roots能直接关联到的对象; 有STW
1.  并发标记︰进行GC Roots Tracing的过程

<!---->

3.  重新标记:修正并发标记期间，因程序运行导致标记发生变化的那一部分对象； 有STW
3.  并发清除∶并发回收垃圾对象

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a564d870a3d48f0bef0c99c0191c0c7~tplv-k3u1fbpfcp-zoom-1.image)

-   在初始标记和重新标记两个阶段还是会发生Stop-the-World
-   使用标记清除算法，多线程并发收集的垃圾收集器

<!---->

-   最后的重置线程，指的是清空跟收集相关的数据并重置，为下一次收集做准备
-   优点∶低停顿、并发执行

<!---->

-   缺点︰并发执行，对CPU资源压力大;

无法处理在处理过程中产生的垃圾，可能导致FullGC

采用的标记清除算法会导致大量碎片，从而在分配大对象是可能触发FullGC

-   开启:-XX:UseConcMarkSweepGC∶使用ParNew +CMS + Serial Old的收集器组合，Serial Old将作为CMS出错的后备收集器
-   -XX:CMSInitiatingOccupancyFraction:设置CMS收集器在老年代空间被使用多少后触发回收，默认80%。因为在第二阶段和第四阶段会不断产生新垃圾，而这些垃圾只会留到下一次清理，所以需要预留空间给这些垃圾。这个参数设置得太高， CMS运行期间预留的内存无法满足程序分配新对象的需要，就会出现一次“并发失败”（Concurrent Mode Failure），这时候虚拟机将不 得不启动后备预案：冻结用户线程的执行，临时启用Serial Old收集器来重新进行老年代的垃圾收集， 但这样停顿时间就很长了。 

## G1收集器

G1是一款面向服务端应用的收集器，与其它收集器相比，具有如下特点∶

1.  G1把内存划分成多个独立的区域(Region)
1.  G1仍采用分代思想，保留了新生代和老年代，但它们不再是物理隔离的，而是一部分Region的集合，且不需要Region是连续的

<!---->

3.  Region中还有一类特殊的Humongous区域，专门用来存储大对象。G1认为只要大小超过了一个 Region容量一半的对象即可判定为大对象。每个Region的大小可以通过参数-XX：G1HeapRegionSize设 定，取值范围为1MB～32MB，且应为2的N次幂。而对于那些超过了整个Region容量的超级大对象， 将会被存放在N个连续的Humongous Region之中，G1的大多数行为都把Humongous Region作为老年代 的一部分来进行看待 
3.  G1为每一个Region设 计了两个名为TAMS（Top at Mark Start）的指针，把Region中的一部分空间划分出来用于并发回收过 程中的新对象分配，并发回收时新分配的对象地址都必须要在这两个指针位置以上 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e81901dce1845ef86fb4119250486ee~tplv-k3u1fbpfcp-zoom-1.image)

3.  G1能充分利用多CPU、多核环境硬件优势，尽量缩短STw
3.  G1整体上采用标记-整理算法，局部是通过复制算法，不会产生内存碎片

<!---->

5.  G1的停顿可预测，能明确指定在一个时间段内，消耗在垃圾收集上的时间不能超过多长时间， G1收集器之所以能建立可预测的停顿时间模型，是因为**它将Region作为单次回收的最小单元，即每次收集到的内存空间都是Region大小的整数倍**，这样可以有计划地避免 在整个Java堆中进行全区域的垃圾收集 
5.  **G1跟踪各个Region里面垃圾堆的价值大小，在后台维护一个优先列表，每次根据允许的时间来回收价值最大的区域，从而保证在有限时间内的高效收集**

**步骤：**

跟CMS类似，也分为四个阶段∶

1.  初始标记∶只标记GCRoots能直接关联到的对象
1.  并发标记∶进行GC Roots Tracing的过程

<!---->

3.  最终标记∶修正并发标记期间，因程序运行导致标记发生变化的那一部分对象
3.  筛选回收:根据时间来进行价值最大化的回收

除第二阶段以外，其他阶段都需要STW

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1d69ff94a2a41009038fcd56444e9b2~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a81803d8758d43219300f3c25e84f61c~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28075f1e057d40f0bcb4651d5594f494~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37682e641a0841b285ca53b6f9487ce6~tplv-k3u1fbpfcp-zoom-1.image)


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f7a3c07632448c28e51ef85c48a953f~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17bf0663aeef41879be9144d15b4e0a4~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/604ee3f49392491e9bfdf2b8b6575344~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b728be77a9f44c9ae378a8663cf7d57~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b97045ba62404f5ca57a3d9406491383~tplv-k3u1fbpfcp-zoom-1.image)

-   使用和配置G1: -XX:+UseG1GC:开启G1，JDK13默认就是G1
-   -XX:MaxGCPauseMillis=n:最大GC停顿时间，这是个软目标，JVM将尽可能（但不保证）停顿小于这个时间

-XX:InitiatingHeapOccupancyPercent=n:堆占用了多少的时候就触发GC，默认为45

G1收集器的缺点：

1.  每个Region都维护有自己的记忆集 ， G1至少要耗费大约相当于Java堆容量10%至20%的额 外内存来维持收集器工作。 
1.

## 低延迟垃圾收集器 （待办）

## GC性能指标

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8da165acfb254b318b3d27d9219914d2~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2cb57bee4c84223be54e08abfa72e48~tplv-k3u1fbpfcp-zoom-1.image)



## JVM配置内存原则

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c47ffe5f99f49c8b2ee7aa78fbde990~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd14205b1c3c4316ac52489ad5f85ac7~tplv-k3u1fbpfcp-zoom-1.image)


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4403a1ae82649cf8370365ec17d4720~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e395aff3a8b0426a9b76fda42444597e~tplv-k3u1fbpfcp-zoom-1.image)

