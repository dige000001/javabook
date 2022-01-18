## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

  # 基本概念
  早期程序员会自己通过一种同步器去实现另一种相近的同步器，例如用可重入锁去实现信号量，或反之。这显然不够优雅，于是在 JSR166（java 规范提案）中创建了 AQS，提供了这种通用的同步器机制。它使用了一个int成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作


AQS 要实现的功能目标

-   阻塞版本获取锁 acquire 和非阻塞的版本尝试获取锁 tryAcquire
-   获取锁超时机制

<!---->

-   通过打断取消机制
-   独占机制及共享机制

<!---->

-   条件不满足时的等待机制



每一个基于AQS实现的同步器都会包含两种类型的操作，如下。

-   至少一个acquire操作。这个操作阻塞调用线程，除非/直到AQS的状态允许这个线程继续执行。
-   ·至少一个release操作。这个操作改变AQS的状态，改变后的状态可允许一个或多个阻塞线程被解除阻塞。

# AQS的实现分析

## 同步队列

同步器依赖内部的同步队列(一个FIFO双向队列)来完成同步状态的管理，当前线程获取同步状态失败时，同步器会将当前线程以及等待状态等信息构造成为一个节点(Node)并将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态。

同步队列中的节点(Node)用来保存获取同步状态失败的线程引用、等待状态以及前驱和后继节点，节点的属性类型与名称以及描述如表5-5所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1da275cb4d141c7b7d75fae8669924b~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d13ecedba5b94e6ca0f488583687a2e9~tplv-k3u1fbpfcp-zoom-1.image)

入队是cas操作，而由于出队时的线程占用着锁，所以不必cas

## 独占式同步状态获取与释放（即独占地获得锁）

通过调用同步器的`acquire(int arg)`方法可以获取同步状态，该方法对中断不敏感，也就是由于线程获取同步状态失败后进入同步队列中，后续对线程进行中断操作时，线程不会从同步队列中移出，该方法代码如代码清单5-3所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2723baffbcef42ccbf538f0ad26ad214~tplv-k3u1fbpfcp-zoom-1.image)

上述代码主要完成了同步状态获取、节点构造、加入同步队列以及在同步队列中自旋等待的相关工作，其主要逻辑是：

1.  首先调用自定义同步器实现的`tryAcquire(int arg)`方法，该方法保证线程安全的获取同步状态
1.  如果同步状态获取失败，则构造同步节点（独占式Node.EXCLUSIVE，同一时刻只能有一个线程成功获取同步状态）并通过`addWaiter(Nodenode)`方法将该节点加入到同步队列的尾部，最后调用`acquireQueued(Node node.int arg)`方法，使得该节点以“死循环”的方式获取同步状态。如果获取不到则阻塞节点中的线程，而被阻塞线程的唤醒主要依靠前驱节点的出队或阻塞线程被中断来实现。



独占式同步状态获取流程：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/135a97dacb11442f986a73197aeb1db3~tplv-k3u1fbpfcp-zoom-1.image)

分析了独占式同步状态获取和释放过程后，适当做个总结：在获取同步状态时，同步器维护一个同步队列，获取状态失败的线程都会被加入到队列中并在队列中进行自旋；移出队列（或停止自旋）的条件是前驱节点为头节点且成功获取了同步状态。在释放同步状态时，同步器调用`tryRelease(int arg)`方法释放同步状态，然后唤醒头节点的后继节点。

## 共享式同步状态获取与释放

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09ddd7c672b44772b99c9152d87dea16~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92777cb3d10f4d83a57424245fa607a2~tplv-k3u1fbpfcp-zoom-1.image)

在`acquireShared(int arg)`方法中，同步器调用`tryAcquireShared(int arg)`方法尝试获取同步状态，`tryAcquireShared(int arg)`方法返回值为int类型，当返回值大于等于0时，表示能够获取到同步状态。因此，在共享式获取的自旋过程中，成功获取到同步状态并退出自旋的条件就是`tryAcquireShared(int arg)`方法返回值大于等于O。可以看到，在`doAcquireShared(int arg)`方法的自旋过程中，如果当前节点的前驱为头节点时，尝试获取同步状态，如果返回值大于等于0，表示该次获取同步状态成功并从自旋过程中退出。

注：共享式释放同步状态之后，将会唤醒后续处于等待状态的节点。对于能够支持多个线程同时访问的并发组件（比如Semaphore)，它和独占式主要区别在于`tryReleaseShared(int arg)`方法必须确保同步状态（或者资源数）线程安全释放，一般是通过循环和CAS来保证的，因为释放同步状态的操作会同时来自多个线程。

## 独占式超时获取同步状态

从图5-7中可以看出，独占式超时获取同步状态`doAcquireNanos(int arg.long nanosTimeout)`和独占式获取同步状态`acquire(int args)`在流程上非常相似，其主要区别在于未获取到同步状态时的处理逻辑。acquire(int args)在未获取到同步状态时，将会使当前线程一直处于等待状态，而`doAcquireNanos(int arg.Jong nanosTimeout)`会使当前线程等待nanosTimeout纳秒，如果当前线程在nanosTimeout纳秒内没有获取到同步状态，将会从等待逻辑中自动返回。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/595fb513f6e644a796621c937122822f~tplv-k3u1fbpfcp-zoom-1.image)

# Reentrantlock

## 非公平锁实现原理

### 加锁解锁流程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ded2f037e3f145e195fe47e5726872ec~tplv-k3u1fbpfcp-zoom-1.image)

NonfairSync 继承自 AQS

**流程：**

#### 1. 没有竞争时

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90d4be0c3feb4f0c93861e9420260a22~tplv-k3u1fbpfcp-zoom-1.image)



#### 2. 第一个竞争出现时

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/221957d20b0b4c0984e95798557c5bac~tplv-k3u1fbpfcp-zoom-1.image)

Thread-1 执行了：

1. CAS 尝试将 state 由 0 改为 1，结果失败

2. 进入 tryAcquire 逻辑，这时 state 已经是1，结果仍然失败

#### 3. 接下来进入 addWaiter 逻辑，构造 Node 队列

-   图中黄色三角表示该 Node 的 waitStatus 状态，其中 0 为默认正常状态
-   Node 的创建是懒惰的

<!---->

-   其中第一个 Node 称为 Dummy（哑元）或哨兵，用来占位，并不关联线程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9dfd0903a0de4fc59e34e5cae4ba5056~tplv-k3u1fbpfcp-zoom-1.image)

#### 4. 当前线程进入 acquireQueued 逻辑：

1. acquireQueued 会在一个死循环中不断尝试获得锁，失败后进入 park 阻塞

2. 如果自己是紧邻着 head（排第二位），那么再次 tryAcquire 尝试获取锁，当然这时 state 仍为 1，失败

3. 进入 shouldParkAfterFailedAcquire 逻辑，将前驱 node，即 head 的 waitStatus 改为 -1，这次返回 false

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1343e9b066244be8ab49ee56e26bb54~tplv-k3u1fbpfcp-zoom-1.image)

4. shouldParkAfterFailedAcquire 执行完毕回到 acquireQueued ，再次 tryAcquire 尝试获取锁，当然这时state 仍为 1，失败

5. 当再次进入 shouldParkAfterFailedAcquire 时，这时因为其前驱 node 的 waitStatus 已经是 -1，这次返回true

6. 进入 parkAndCheckInterrupt， Thread-1 `park`（灰色表示）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8018102414545bea515fa0acf41f422~tplv-k3u1fbpfcp-zoom-1.image)

再次有多个线程经历上述过程竞争失败，变成这个样子

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a51e883654694e809ffb8931ae597e96~tplv-k3u1fbpfcp-zoom-1.image)

#### 5. Thread-0 释放锁，进入 tryRelease 流程，如果成功

-   设置 exclusiveOwnerThread 为 null
-   state = 0

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c712e540ebe14586b4cea2506c7c33e6~tplv-k3u1fbpfcp-zoom-1.image)

-   当前队列不为 null，并且 head 的 waitStatus = -1，进入 unparkSuccessor 流程，找到队列中离 head 最近的一个 Node（没取消的），unpark 恢复其运行，本例中即为 Thread-1




#### 6. 回到 Thread-1 的 acquireQueued 流程

如果加锁成功（没有竞争），会设置

-   exclusiveOwnerThread 为 Thread-1，state = 1
-   head 指向刚刚 Thread-1 所在的 Node，该 Node 清空 Thread

<!---->

-   原本的 head 因为从链表断开，而可被垃圾回收

注：此时若有其他锁来竞争，Thread-1可能失败，继续park

加锁解锁源码见《原理》附件

## 可重入原理

源码见《原理》附件

## 可打断原理

源码见《原理》附件

对于不可打断模式，在此模式下，即使它被打断，仍会驻留在 AQS 队列中，一直要等到获得锁后方能得知自己被打断了

对于可打断模式，一被打断就抛出异常

## 公平锁实现原理

源码见《原理》附件

与非公平锁主要区别在于 tryAcquire 方法的实现，公平锁先检查 AQS 队列中是否有前驱节点, 没有才去竞争

## 条件变量实现原理

源码见《原理》附件

每个条件变量其实就对应着一个等待队列，其实现类是 ConditionObject

### park流程：

1.  开始 Thread-0 持有锁，调用 await，进入 ConditionObject 的 addConditionWaiter 流程，创建新的 Node 状态为 -2（Node.CONDITION），关联 Thread-0，加入等待队列尾部

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b6fb56460b348b6b4e3aec5ad1ae15e~tplv-k3u1fbpfcp-zoom-1.image)

2.  接下来进入 AQS 的 fullyRelease 流程，释放同步器上的锁，unpark AQS 队列中的下一个节点，竞争锁，假设没有其他竞争线程，那么 Thread-1 竞争成功，同时park 阻塞 Thread-0

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b5c82bc59a4a433fbb4c155596f4e2cf~tplv-k3u1fbpfcp-zoom-1.image)



### signal 流程

1.  假设 Thread-1 要来唤醒 Thread-0，进入 ConditionObject 的 doSignal 流程，取得等待队列中第一个 Node，即 Thread-0 所在 Node。
1.  执行 transferForSignal 流程，将该 Node 加入 AQS 队列尾部，将 Thread-0 的 waitStatus 改为 0，Thread-3 的waitStatus 改为 -1

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75890c54bba94ec3862eeba820af5ade~tplv-k3u1fbpfcp-zoom-1.image)

3.  Thread-1 释放锁，进入 unlock 流程，略


# 读写锁原理

读写锁用的是同一个 Sycn 同步器，因此等待队列、state 等也是同一个

t1 w.lock，t2 r.lock:

1.  t1 成功上锁，流程与 ReentrantLock 加锁相比没有特殊之处，不同是写锁状态占了 state 的低 16 位，而读锁

使用的是 state 的高 16 位【写进程上锁】

2.  t2 执行 r.lock，这时进入读锁的 sync.acquireShared(1) 流程，首先会进入 tryAcquireShared 流程。如果有写

锁占据，那么 tryAcquireShared 返回 -1 表示失败

tryAcquireShared 返回值表示

-   -1 表示失败
-   0 表示成功，但后继节点不会继续唤醒

<!---->

-   正数表示成功，而且数值是还有几个后继节点需要唤醒，读写锁返回 1

3.  这时会进入 sync.doAcquireShared(1) 流程，首先也是调用 addWaiter 添加节点，不同之处在于节点被设置为

Node.SHARED 模式而非 Node.EXCLUSIVE 模式，注意此时 t2 仍处于活跃状态

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53265cfaecc14e21ad9a9a701d8da256~tplv-k3u1fbpfcp-zoom-1.image)

4.  t2 会看看自己的节点是不是老二，如果是，还会再次调用 tryAcquireShared(1) 来尝试获取锁
4.  如果没有成功，在 doAcquireShared 内 for (;;) 循环一次，把前驱节点的 waitStatus 改为 -1，再 for (;;) 循环一次尝试 tryAcquireShared(1) 如果还不成功，那么在 parkAndCheckInterrupt() 处 park 【以上几条为读进程上锁】

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bd602121cc5419886156400e5a543bb~tplv-k3u1fbpfcp-zoom-1.image)

6.  t1 w.unlock这时会走到写锁的 sync.release(1) 流程，调用 sync.tryRelease(1) 成功，将exclusiveOwnerThread设为NULL，然后执行唤醒流程 sync.unparkSuccessor 【写进程解锁】
6.  接下来执行唤醒流程 sync.unparkSuccessor，即让老二恢复运行，这时 t2 在 doAcquireShared 内parkAndCheckInterrupt() 处恢复运行，这回再来一次 for (;;) 执行 tryAcquireShared 成功则让读锁计数加一

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9fddc23e6f8e4d0795cd487c5af7285f~tplv-k3u1fbpfcp-zoom-1.image)

8.  这时 t2 已经恢复运行，接下来 t2 调用 setHeadAndPropagate(node, 1)，它原本所在节点被置为头节点，在 setHeadAndPropagate 方法内还会检查下一个节点是否是 shared，如果是则调用doReleaseShared() 将 head 的状态从 -1 改为 0 并唤醒老二，这时 t3 在 doAcquireShared 内parkAndCheckInterrupt() 处恢复运行

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0eaebcff7fa548dbb8af78d2af9e5223~tplv-k3u1fbpfcp-zoom-1.image)

9.  这回T3再来一次 for (;;) 执行 tryAcquireShared 成功则让读锁计数加一

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee95be8233a44dc29b7aedb25916a540~tplv-k3u1fbpfcp-zoom-1.image)

10. 重复以上，直到t4不是共享读线程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88b7b68e6da945f6bc08b6c413cee3b9~tplv-k3u1fbpfcp-zoom-1.image)

11. t2 进入 sync.releaseShared(1) 中，调用 tryReleaseShared(1) 让计数减一，但由于计数还不为零，t3 进入 sync.releaseShared(1) 中，调用 tryReleaseShared(1) 让计数减一，这回计数为零了，进入doReleaseShared() 将头节点从 -1 改为 0 并唤醒老二，即T4 【读进程解锁】
11. 之后 t4 在 acquireQueued 中 parkAndCheckInterrupt 处恢复运行，再次 for (;;) 这次自己是老二，并且没有其他竞争，tryAcquire(1) 成功，修改头结点，流程结束

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ae6d7bb17ec4c4db64062d35cae7a5a~tplv-k3u1fbpfcp-zoom-1.image)

