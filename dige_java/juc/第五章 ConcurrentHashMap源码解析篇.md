## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂
# 死链

注：发生在jdk7 concurrentHashMap扩容的时候

在jdk7的concurrentHashMap(简称CHM)中，新添加的节点会添加在数组下标的链表头部，而在jdk8中，会添加在数组下标的链表尾部。

扩容源码：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51908b2703c54ccf8bd630a5f61a253b~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9973c5736d1e45a099c61694b2520a0c~tplv-k3u1fbpfcp-zoom-1.image)

锐评：

究其原因，是因为在多线程环境下使用了非线程安全的 map 集合

JDK 8 虽然将扩容算法做了调整，不再将元素加入链表头（而是保持与扩容前一样的顺序），但仍不意味着能

够在多线程环境下能够安全扩容，还会出现其它问题（如扩容丢数据）

jdk7: map包含16个segment，每个segement维护一个hashentry数组(即多个链表，每个数组节点就是一个链表)，扩容时对该segment上锁，对hashentry数组扩容

优点：如果多个线程访问不同的 segment，实际是没有冲突的，这与 jdk8 中是类似的

缺点：Segments 数组默认大小为16，这个容量初始化指定后就不能改变了，并且不是懒惰初始化

jdk8：map包含一个数组，每个数组节点都是一个链表，扩容时对该数组节点加锁，要扩容的是整个数组而不是链表

# JDK 8 ConcurrentHashMap

## 重要属性和内部类

```
// 默认为 0
// 当初始化时, 为 -1
// 当扩容时, 为 -(1 + 扩容线程数)
// 当初始化或扩容完成后，为 下一次的扩容的阈值大小
private transient volatile int sizeCtl;

// 整个 ConcurrentHashMap 就是一个 Node[]
static class Node<K,V> implements Map.Entry<K,V> {}

// hash 表
//采用table数组元素作为锁，从而实现了对每一行数据进行加锁，进一步减少并发冲突的概率。
transient volatile Node<K,V>[] table;

// 扩容时的 新 hash 表
private transient volatile Node<K,V>[] nextTable;

// 扩容时如果某个 bin 迁移完毕, 用 ForwardingNode 作为旧 table bin 的头结点
static final class ForwardingNode<K,V> extends Node<K,V> {}

// 用在 compute 以及 computeIfAbsent 时, 用来占位, 计算完成后替换为普通 Node
static final class ReservationNode<K,V> extends Node<K,V> {}

// 作为 treebin 的头节点, 存储 root 和 first
static final class TreeBin<K,V> extends Node<K,V> {}

// 作为 treebin 的节点, 存储 parent, left, right
static final class TreeNode<K,V> extends Node<K,V> {}
```

## 重要方法

```
// 获取 Node[] 中第 i 个 Node
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i)
    
// cas 修改 Node[] 中第 i 个 Node 的值, c 为旧值, v 为新值
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i, Node<K,V> c, Node<K,V> v)
    
// 直接修改 Node[] 中第 i 个 Node 的值, v 为新值
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v)
```

## 构造器分析

可以看到实现了懒惰初始化，在构造方法中仅仅计算了 table 的大小，以后在第一次使用时才会真正创建

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccc9f8a0306d41c6ba3f1ef0f9cebe9a~tplv-k3u1fbpfcp-zoom-1.image)

## get 流程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e562846ce2874361aa4afe082e15af4a~tplv-k3u1fbpfcp-zoom-1.image)

## put 流程

代码见附件《原理》部分

参考：<https://www.yuque.com/normalgamer/msmcvb/dvqgxa#L4tpt>

`int hash = spread(key.hashCode()); `

`f = tabAt(tab, i = (n - 1) & hash)) `

hash和定位




总结一下：

详细版：

1.  判断put进来的key和value是否为null，如果为null抛异常。（ConcurrentHashMap的key、value不能为null）。
1.  随后进入无限循环(没有判断条件的for循环)，何时插入成功，何时退出。

<!---->

3.  在无限循环中，若table数组为空（底层数组加链表），则调用initTable()，初始化table；
3.  若table不为空，先hashCode(),再无符号右移16位异或，再(n-1)&hash，定位到table中的位置，如果该位置为空（说明还没有发生哈希冲突），则使用CAS将新的节点放入table中。

<!---->

5.  如果该位置不为空，且该节点的hash值为MOVED（即为forward节点，哈希值为-1，其中含有指向nextTable的指针，class ForwardingNode中有nexttable变量），说明此时正在扩容，且该节点已经扩容完毕，如果还有剩余任务（任务没分配完）该线程执行helpTransfer方法，帮助其他线程完成扩容，如果已经没有剩余任务，则该线程可以直接操作新数组nextTable进行put。
5.  如果该位置不为空，且该节点不是forward节点。对桶中的第一个结点（即table表中的结点，哈希值相同的链表的第一个节点）进行加锁（锁是该结点，如果此时还有其他线程想来put，会阻塞）（如果不加锁，可能在遍历链表的过程中，又有其他线程放进来一个相同的元素，但此时我已经遍历过，发现没有相同的，这样就会产生两个相同的），对该桶进行遍历，桶中的结点的hash值与key值与给定的hash值和key值相等，则根据标识选择是否进行更新操作（用给定的value值替换该结点的value值），若遍历完桶仍没有找到hash值与key值和指定的hash值与key值相等的结点，则直接新生一个结点并赋值为之前最后一个结点的下一个结点。

<!---->

7.  若binCount值达到红黑树转化的阈值，则将桶中的结构转化为红黑树存储，最后，增加binCount的值。最后调用addcount方法，将concurrenthashmap的size加1，调用size()方法时会用到这个值。

简单版：

循环：

1.  查看链表创建没有，没有就创建
1.  查看该链表有头结点吗？没有就用该要插入的节点做头节点，然后break结束函数

<!---->

3.  查看是否在扩容，若是，就帮忙扩容
3.  将节点加入链表or红黑树，取决于该数组下标的东西是链表还是红黑树

<!---->

5.  如果加入节点后链表长度大于树化阈值(默认8)，就将链表转化为红黑树(当删除操作使得长度小于退化阈值(默认6)，则会变回链表)




## 扩容

代码见附件《原理》

参考：<https://www.yuque.com/normalgamer/msmcvb/dvqgxa#bAYl5>

整个扩容操作分为两个部分：

第一部分是构建一个nextTable,它的容量是原来的两倍，这个操作是单线程完成的。

第二个部分就是将原来table中的元素复制到nextTable中，这里允许多线程进行操作。

其他线程调用helptransfer方法来协助扩容时，首先拿到nextTable数组，再调用transfer方法。给新来的线程分配任务（默认是16个桶一个任务）。

遍历自己所分到的桶：

1、桶中元素不存在，则通过CAS操作设置桶中第一个元素为ForwardingNode，其Hash值为MOVED（-1）,同时该元素含有新的数组引用

此时若其他线程进行put操作，发现第一个元素的hash值为-1则代表正在进行扩容操作（并且表明该桶已经完成扩容操作了，可以直接在新的数组中重新进行hash和插入操作），该线程就可以去帮助扩容，或者没有任务则不用参与，此时可以去直接操作新的数组了

2、桶中元素存在且hash值为-1，则说明该桶已经被处理了（本不会出现多个线程任务重叠的情况，这里主要是该线程在执行完所有的任务后会再次进行检查，再次核对）

3、桶中为链表或者红黑树结构，则需要获取桶锁，防止其他线程对该桶进行put操作，然后处理方式同HashMap的处理方式一样，对桶中元素分为2类，分别代表当前桶中和要迁移到新桶中的元素。设置完毕后代表桶迁移工作已经完成，旧数组中该桶可以设置成ForwardingNode了，**已经完成从table复制到nextTable的节点，要设置为forward**

## size 计算流程

代码见附件《原理》部分

size 计算实际发生在 put，remove 改变集合元素的操作之中

-   没有竞争发生，向 baseCount 累加计数
-   有竞争发生，新建 counterCells，向其中的一个 cell 累加计数

<!---->

-   -   counterCells 初始有两个 cell
    -   如果计数竞争比较激烈，会创建新的 cell 来累加计数




## 总结

Java 8 数组（Node） +（ 链表 Node | 红黑树 TreeNode ） 以下数组简称（table），链表简称（bin）

-   初始化，使用 cas 来保证并发安全，懒惰初始化 table
-   树化，当 table.length < 64 时，先尝试扩容，超过 64 时，并且 bin.length > 8 时，会将链表树化，树化过程会用 synchronized 锁住链表头

<!---->

-   put，如果该 bin 尚未创建，只需要使用 cas 创建 bin；如果已经有了，锁住链表头进行后续 put 操作，元素添加至 bin 的尾部
-   get，无锁操作仅需要保证可见性，扩容过程中 get 操作拿到的是 ForwardingNode 它会让 get 操作在新table 进行搜索

<!---->

-   扩容，扩容时以 bin 为单位进行，需要对 bin 进行 synchronized，但这时妙的是其它竞争线程也不是无事可做，它们会帮助把其它 bin 进行扩容，扩容时平均只有 1/6 的节点会把复制到新 table 中
-   size，元素个数保存在 baseCount 中，并发时的个数变动保存在 CounterCell[] 当中。最后统计数量时累加即可



源码分析 <http://www.importnew.com/28263.html>



