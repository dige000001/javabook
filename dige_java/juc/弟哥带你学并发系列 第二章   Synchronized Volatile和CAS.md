## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# Monitor

每个 Java 对象都可以关联一个 Monitor 对象，如果使用 synchronized 给对象上锁（重量级）之后，该对象头的

Mark Word 中就被设置指向 Monitor 对象的指针

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d790b3da08c46ea9310e8d9c22fdd47~tplv-k3u1fbpfcp-zoom-1.image)

1.  刚开始 Monitor 中 Owner 为 null
1.  当 Thread-2 执行 synchronized(obj) 就会将 Monitor 的所有者 Owner 置为 Thread-2，Monitor中只能有一个 Owner

<!---->

3.  在 Thread-2 上锁的过程中，如果 Thread-3，Thread-4，Thread-5 也来执行 synchronized(obj)，就会进入EntryList BLOCKED
3.  Thread-2 执行完同步代码块的内容，然后唤醒 EntryList 中等待的线程来竞争锁，竞争锁时是非公平的

图中 WaitSet 中的 Thread-0，Thread-1 是之前获得过锁，但条件不满足进入 WAITING 状态的线程，后面讲

wait-notify 时会分析

Synchronized对于同步块的实现使用了monitorenter和monitorexit指令，而同步方法则是依靠方法修饰符上的ACC_SYNCHRONIZED来完成的。无论采用哪种方式，其本质是对一个对象的监视器(monitor）进行获取，而这个获取过程是排他的，也就是同一时刻只能有一个线程获取到由synchronized所保护对象的监视器。

任意一个对象都拥有自己的监视器，当这个对象由同步块或者这个对象的同步方法调用时，执行方法的线程必须先获取到该对象的监视器才能进入同步块或者同步方法，而没有获取到监视器（执行该方法）的线程将会被阻塞在同步块和同步方法的入口处，进入BLOCKED状态。

# Java 对象头

以 32 位虚拟机为例

### 普通对象

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67fe5dacfe1f4a0aa11bf32447af4af3~tplv-k3u1fbpfcp-zoom-1.image)

### 数组对象

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9527915103de4c469e658b9c83cbce13~tplv-k3u1fbpfcp-zoom-1.image)

\


## MarkWord

### 32位

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/200be315bc774fee9c719de09de86f70~tplv-k3u1fbpfcp-zoom-1.image)

### 64位

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/493d3cc111ff476383cd1ae4d63a4900~tplv-k3u1fbpfcp-zoom-1.image)

\


# sychronized

## 轻量级锁

轻量级锁的使用场景：如果一个对象虽然有多线程要加锁，但加锁的时间是错开的（也就是没有竞争），那么可以使用轻量级锁来优化。

轻量级锁对使用者是透明的，即语法仍然是 `synchronized`

让我们看看锁的工作流程：

1.  当一个线程要使用`synchronized(obj)`时，创建锁记录（Lock Record）对象，每个线程的栈帧都会包含一个锁记录的结构，内部可以存储锁定对象的Mark Word

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c65d296faa443ce82a045f6e3938026~tplv-k3u1fbpfcp-zoom-1.image)

2.  让锁记录中 Object reference 指向锁对象，并尝试用 cas替换 Object 的 Mark Word，将 Mark Word 的值存入锁记录（即交换二者的值）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8983c159fba4f15a7b309a6186498c9~tplv-k3u1fbpfcp-zoom-1.image)

3.  若交换成功，对象头中存储了`锁记录地址和状态00`，表示由该线程给对象加锁

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28fc11e2911447a6b90f33c89e2a2d1e~tplv-k3u1fbpfcp-zoom-1.image)

4.  如果 cas 失败，有两种情况：

-   -   如果是其它线程已经持有了该 Object 的轻量级锁，这时表明有竞争，进入锁膨胀过程
    -   如果是自己执行了 synchronized 锁重入，那么再添加一条 Lock Record 作为重入的计数

若发生锁重入：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d7fa784e52314d4aad1406647eab4330~tplv-k3u1fbpfcp-zoom-1.image)

5.  当退出 synchronized 代码块（解锁时）如果有取值为 null 的锁记录，表示有重入，这时重置锁记录，表示重入计数减一

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4304fa4e91704a5badbe478705177902~tplv-k3u1fbpfcp-zoom-1.image)

6.  当退出 synchronized 代码块（解锁时）锁记录的值不为 null，这时使用 cas 将 Mark Word 的值恢复给对象头

-   -   成功，则解锁成功
    -   失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁解锁流程

## 锁膨胀

如果在尝试加轻量级锁的过程中，CAS 操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有竞争），这时需要进行锁膨胀，将轻量级锁变为重量级锁。

流程如下：

1.  当 Thread-1 进行轻量级加锁时，Thread-0 已经对该对象加了轻量级锁，这时 Thread-1 加轻量级锁失败，进入锁膨胀流程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a64b89b98bf64f25b56fa3c87547e6be~tplv-k3u1fbpfcp-zoom-1.image)

2.  Thread-1为 Object 对象申请 Monitor 锁，让 Object 指向重量级锁地址，然后自己进入 Monitor 的 EntryList BLOCKED

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4220dadd67744499bf5bfd6189f35b6~tplv-k3u1fbpfcp-zoom-1.image)

3.  当 Thread-0 退出同步块解锁时，使用 cas 将 Mark Word 的值恢复给对象头，失败。这时会进入重量级解锁流程，即按照 Monitor 地址找到 Monitor 对象，设置 Owner 为 null，唤醒 EntryList 中 BLOCKED 线程



## 自旋优化

所谓自旋，就是当竞争锁发现锁已经被其他线程占用时，会再尝试3次

重量级锁竞争的时候，还可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程就可以避免阻塞

-   自旋会占用 CPU 时间，单核 CPU 自旋就是浪费，多核 CPU 自旋才能发挥优势。
-   在 Java 6 之后自旋锁是自适应的，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会高，就多自旋几次；反之，就少自旋甚至不自旋，总之，比较智能。

<!---->

-   Java 7 之后不能控制是否开启自旋功能



## 偏向锁

轻量级锁在没有竞争时（就自己这个线程），每次重入仍然需要执行 CAS 操作。

Java 6 中引入了偏向锁来做进一步优化：只有第一次使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f2126780fde41c7af34a3f2e099eefb~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ea4c16fdc9a4138ad898a1448419d57~tplv-k3u1fbpfcp-zoom-1.image)

回忆一下对象头格式

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5fbda2a2cd174aa3bf39b654f3f7ea3c~tplv-k3u1fbpfcp-zoom-1.image)

一个对象创建时：

-   如果开启了偏向锁（默认开启），那么对象创建后，markword 值为 0x05 即最后 3 位为 101，这时它的thread、epoch、age 都为 0

<!---->

-   偏向锁是默认是延迟的，不会在程序启动时立即生效，如果想避免延迟，可以加 VM 参数

`-XX:BiasedLockingStartupDelay=0` 来禁用延迟

-   如果没有开启偏向锁，那么对象创建后，markword 值为 0x01 即最后 3 位为 001，这时它的 hashcode、age 都为 0，第一次用到 hashcode 时才会赋值


P.S:处于偏向锁的对象解锁后，线程 id 仍存储于对象头中，且MarkWord后面三位依旧是101

### 撤销 - 调用对象 hashCode

调用了对象的 hashCode，但偏向锁的对象 MarkWord 中存储的是线程 id，如果调用 hashCode 会导致偏向锁被撤销

-   轻量级锁会在锁记录中记录 hashCode
-   重量级锁会在 Monitor 中记录 hashCode



### 撤销 - 其它线程使用对象

当有其它线程使用偏向锁对象时，会将偏向锁升级为轻量级锁



### 撤销 - 调用 wait/notify

会升级为重量级锁，因为这两个函数只有重量级锁才有

###

### 批量重偏向

如果对象虽然被多个线程访问，但没有竞争，这时偏向了线程 T1 的对象仍有机会重新偏向 T2，重偏向会重置对象的 Thread ID

当撤销偏向锁阈值超过 20 次后，jvm 会这样觉得，我是不是偏向错了呢，于是会在给这些对象加锁时重新偏向至加锁线程。（疑问：是所有对象还是某个类的所有对象？）



### 批量重撤销

当撤销偏向锁阈值超过 40 次后，jvm 会这样觉得，自己确实偏向错了，根本就不该偏向。于是整个类的所有对象都会变为不可偏向的，新建的对象也是不可偏向的

# Volatile

Volatile保证可见性，在工作内存中，每次使用volatile变量都必须先从主内存刷新最新值，使用后立刻同步回主内存（volatile变量依然有工作内存的拷贝，但由于它特殊的操作顺序（每次使用前都需要刷新），所以看起来如同直接在主内存中读写访问一般）（将use和load动作相关联，即use前必须是load，load后必须是use；将assign和store相关联，即store前必须是assign，assign后必须是store）

Volatile保证有序性（禁止指令重排序优化，jkd1.5以后重新修复）：先行先发生原则中volatile变量规则：对一个volatile变量的写操作先行发生于（能被后面的读操作感知到）**后面（时间上）** 对这个变量的读操作。通过内存屏障（Memory Barrier）实现，指令重排序时不能把后面的指令重排序到内存屏障之前的位置。

Volatile不能保证原子性。如果我们对一个volatile修饰的变量进行多线程下的自增操作，还是会出现线程安全问题。根本原因在于volatile关键字无法对自增进行安全性修饰，因为自增分为三步，读取、+1、写入。中间多个线程同时执行+1操作，还是会出现线程安全性问题。

①volatile轻量级，只能修饰变量。synchronized重量级，还可修饰方法

②volatile只能保证数据的可见性，不能用来同步，因为多个线程并发访问volatile修饰的变量不会阻塞。（常用于循环判断标志）

synchronized不仅保证可见性，而且还保证原子性，因为，只有获得了锁的线程才能进入临界区（操作共享变量的代码段），从而保证临界区中的所有语句都全部执行。多个线程争抢synchronized锁对象时，会出现阻塞。

volatile可见性（use和load、assign和store），有序性（禁止重排序，happens-before，内存屏障）

原子性：synchronized

可见性：volatile、synchronized、final

有序性：volatile、synchronized（相当于保证只有一个线程执行代码段，保证有序性）

# CAS **(UnSafe包)**

java.util.concurrent包完全建立在CAS之上的

CAS是一条CPU的原子指令，其实现方式是基于硬件平台的汇编指令，就是说CAS是靠硬件实现的。

CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。

拿出AtomicInteger来研究在没有锁的情况下是如何做到数据正确性的。`private volatile int value;`

在没有锁的机制下可能需要借助volatile原语，保证线程间的数据是可见的（共享的）。这样才获取变量的值的时候才能直接读取。

```
public final int incrementAndGet() {
    for (;;) {//无限循环直到成功
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))//相同返回true，不同返回false
            return next;
    }
}

而compareAndSet利用JNI来完成CPU指令的操作。
public final boolean compareAndSet(int expect, int update) {   
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
```

## CAS带来的3个问题

1.  ABA问题。因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就会变成1A-2B－3A。\
    从Java1.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。
1.  循环时间长开销大。自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

<!---->

3.  只能保证一个共享变量的原子操作。当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。

