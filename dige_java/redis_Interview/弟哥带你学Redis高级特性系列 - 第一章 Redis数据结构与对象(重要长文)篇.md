---
theme: qklhk-chocolate
---
# 字符串

Redis的字符串以c字符串为基础，在其基础上构建了名为simple dynamic string的对象，SDS就是redis使用的字符串类型。以C字符串为基础的好处是可以重用C的某些库。

在Redis里面，C字符串只会作为字符串字面量( string literal）用在一些无须对字符串值进行修改的地方，比如打印日志:

redisLog(REDIS_WARNING,"Redis is now ready to exit,bye bye. . .");

当Redis需要的不仅仅是一个字符串字面量，而是一个可以被修改的字符串值时，Redis就会使用SDS来表示字符串值，比如在Redis的数据库里面，包含字符串值的键值对在底层都是由SDS实现的。

redis>SET msg "hello world"

那么Redis将在数据库中创建一个新的键值对，其中:

1.  键值对的键是一个字符串对象，对象的底层实现是一个保存着字符串"msg"的SDS
2.  键值对的值也是一个字符串对象，对象的底层实现是一个保存着字符串"hello world”的SDS.

## SDS的定义

每个sds.h/ sdshdr结构表示一个SDS值:

```
struct sdshdr {
    
//记录buf数组中已使用字节的数量
//等于SDS所保存字符串的长度
int len;
    
//记录buf数组中未使用字节的数量
int free;
    
//字节数组，用于保存字符串
char buf [];
};
```

SDS遵循C字符串以空字符结尾的惯例，保存空字符的1字节空间不计算在SDS的len属性里面，并且为空字符分配额外的1字节空间，以及添加空字符到字符串末尾等操作，都是由SDS函数自动完成的，所以这个空字符对于SDS的使用者来说是完全透明的。遵循空字符结尾这一惯例的好处是，SDS可以直接重用一部分C字符串函数库里面的函数。

## SDS的特点




**SDS设计有以下几个特点：**

1.  C字符串是不安全的，如对字符数组增加一些字符时，可能会因为程序员忘记分配空间而导致溢出，而SDS的API需要对SDS进行修改时，API 会先检查SDS的空间是否满足修改所需的要求，如果不满足的话，API会自动将SDS的空间扩展至执行修改所需的大小，然后才执行实际的修改操作，所以使用SDS既不需要手动修改SDS的空间大小，也不会出现前面所说的缓冲区溢出问题。

<!---->

2.  **SDS不仅会给要修改的字符串分配足够的空间，还会分配一些free空间，这样设计的好处是以后要修改字符串时，不用每次都分配空间，可以使用已分配的未使用的空间**。具体的修改如下：1. 若修改之后的字符串长度小于1MB，就分配与字符串同样大小的空间；2. 否则，分配1MB空间。

<!---->

3.  获得字符串长度的时间复杂度为O(1)，因为SDS维护着一个len变量。

<!---->

4.  惰性空间释放，惰性空间释放用于优化SDS 的字符串缩短操作：当SDS的API需要缩短SDS保存的字符串时，程序并不立即使用内存重分配来回收缩短后多出来的字节，而是使用free属性将这些字节的数量记录起来，并等待将来使用。与此同时，SDS 也提供了相应的API，让我们可以在有需要时，真正地释放SDS的未使用空间，所以不用担心惰性空间释放策略会造成内存浪费。

<!---->

5.  Redis的SDS是二进制安全的，字符串里面可以有空字符，而这在C字符串中会被认为是字符串结尾。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b98197d108a54ddbbc18112dc36e5b68~tplv-k3u1fbpfcp-zoom-1.image)

# 链表

链表被广泛用于实现Redis 的各种功能，比如列表键、发布与订阅、慢查询、监视器等。

## Redis链表的定义

### 链表节点

每个链表节点使用一个adlist.h/ listNode结构来表示:

```
typedef struct listNode {
    //前置节点
    struct listNode * prev;

    //后置节点
    struct listNode * next;

    //节点的值
    void * value;
} listNode;
```


### 链表

adlist.h/ list

```
typedef struct list {
    //表头节点
    listNode * head;

    //表尾节点
    listNode * tail;

    //链表所包含的节点数量
    unsigned long len;

    //节点值复制函数
    void *(*dup) (void *ptr);

    //节点值释放函数
    void (*free) (void *ptr);

    //节点值对比函数
    int (*match) (void *ptr,void *key) ;
    
}list;
```

关于C语言中的void指针参考这篇文章：<https://blog.csdn.net/qq_33890670/article/details/79964262>

list结构为链表提供了表头指针head、表尾指针tail，以及链表长度计数器len,而dup、free和 match 成员则是用于实现多态链表所需的类型特定函数:

-   dup函数用于复制链表节点所保存的值;
-   free函数用于释放链表节点所保存的值;

<!---->

-   match函数则用于对比链表节点所保存的值和另一个输入值是否相等。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64ff59211bc24c7a8d132beee456abd9~tplv-k3u1fbpfcp-zoom-1.image)

## 特点

-   **双端：** 链表节点带有prev和 next指针，获取某个节点的前置节点和后置节点的复杂度都是O(1)。
-   **无环：** 表头节点的prev指针和表尾节点的next指针都指向NULL，对链表的访问以NULL为终点。

<!---->

-   **带表头指针和表尾指针：** 通过list结构的head指针和tail指针，程序获取链表的表头节点和表尾节点的复杂度为O(1)。
-   **带链表长度计数器：** 程序使用list结构的len属性来对list持有的链表节点进行计数，程序获取链表中节点数量的复杂度为O(1)。

<!---->

-   **多态：** 链表节点使用void*指针来保存节点值，并且可以通过list结构的dup、free、match 三个属性为节点值设置类型特定函数，所以链表可以用于保存各种不同类型的值。

# 字典

Redis 的字典使用哈希表作为底层实现，一个哈希表里面可以有多个哈希表节点，而每个哈希表节点就保存了字典中的一个键值对。

接下来的三个小节将分别介绍Redis的哈希表节点、哈希表以及字典的实现。

## 定义

### 哈希表节点

哈希表节点使用dictEntry结构表示，每个dictEntry结构都保存着一个键值对:

```
typedef struct dictEntry {
    //键
    void *key;
    
    //值
    union {
        void *val;
        uint64_tu64;
        int64_ts64;
    } v;
    
    //指向下个哈希表节点，形成链表
    struct dictEntry *next;
    
} dictEntry;
```

key属性保存着键值对中的键，而val属性则保存着键值对中的值，其中键值对的值可以是一个指针，或者是一个uint64_t整数，又或者是一个int64_t整数。

next属性是指向另一个哈希表节点的指针，这个指针可以将多个哈希值相同的键值对连接在一次，以此来解决键冲突( collision）的问题。

### 哈希表

Redis字典所使用的哈希表由dict.h/ dictht结构定义:

```
typedef struct dictht {
    //哈希表数组
    dictEntry **table;

    //哈希表大小
    unsigned long size;

    //哈希表大小掩码，用于计算索引值
    //总是等于size-1
    unsigned long sizemask;
    
    //该哈希表已有节点的数量
    unsigned long used;
    
} dictht;
```

table属性是一个数组，数组中的每个元素都是一个指向dict.h/dictEntry结构的指针

sizemask属性的值总是等于size-1，这个属性和哈希值一起决定一个键应该被放到table数组的哪个索引上面。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87c8fa0bce914403b720b9cb9c95114e~tplv-k3u1fbpfcp-zoom-1.image)

**注意：redis哈希表解决哈希冲突的方法是链地址法，且新键在前**

### 字典

Redis中的字典由dict.h/ dict结构表示:

```
typedef struct dict {
	//类型特定函数
    dictType *type;
    
	//私有数据
	void *privdata;
    
	//哈希表
	dictht ht [2];
    
	// rehash索引
	// 当rehash不在进行时，值为-1
	int trehashidx; /* rehashing not in progress'if rehashidx == -l */
}dict;
```

type属性和privdata属性是针对不同类型的键值对，为创建多态字典而设置的:

type属性是一个指向dictType结构的指针，每个dictType结构保存了一簇用于操作特定类型键值对的函数，Redis 会为用途不同的字典设置不同的类型特定函数。

而privdata属性则保存了需要传给那些类型特定函数的可选参数。

```
typedef struct dictType {
    
//计算哈希值的函数
unsigned int (*hashFunction) (const void *key) ;
    
//复制键的函数
void * (*keyDup) (void *privdata,const void *key);
    
//复制值的函数
void * ( *valDup) (void *privdata,const void *obj);
    
//对比键的函数
int (*keyCompare)(void *privdata,const void *key1,const void *key2) ;
    
//销毁键的函数
void (*keyDestructor) (void *privdata, void *key) ;
    
//销毁值的函数
void (*valDestructor) (void *privdata, void *obj) ;
    
}dictType;
```

ht属性是一个包含两个项的数组，数组中的每个项都是一个dictht哈希表，一般情况下，字典只使用ht [0]哈希表，ht[1]哈希表只会在对ht [0]哈希表进行rehash时使用。

除了ht[1]之外，另一个和rehash有关的属性就是rehashidx，它记录了rehash目前的进度，如果目前没有在进行rehash，那么它的值为-1。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/edfe56f778b148dfad59667ffc95e29a~tplv-k3u1fbpfcp-zoom-1.image)

## 哈希算法

当要将一个新的键值对添加到字典里面时，程序需要先根据键值对的键计算出哈希值和索引值，然后再根据索引值，将包含新键值对的哈希表节点放到哈希表数组的指定索引上面。

Redis计算哈希值和索引值的方法如下:

1.  使用字典设置的哈希函数，计算键key的哈希值：hash = dict->type->hashFunction (key) ;
1.  使用哈希表的sizemask属性和哈希值，计算出索引值：index=hash & dict->ht[x].sizemask; 根据情况不同，ht[x]可以是ht[0]或者ht [1] 

## 扩容/Rehash

当以下条件中的任意一个被满足时，程序会自动开始对哈希表执行扩展操作:

1.  服务器目前没有在执行BGSAVE命令或者BGREWRITEAOF命令，并且哈希表的负载因子大于等于1。
1.  服务器目前正在执行BGSAVE命令或者BGREWRITEAOF命令，并且哈希表的负载因子大于等于5。

其中哈希表的负载因子可以通过公式:

#负载因子=哈希表已保存节点数量/哈希表大小

load factor = ht[0].used / ht[0].size

根据BGSAVE命令或BGREWRITEAOF命令是否正在执行，服务器执行扩展操作所需的负载因子并不相同，这是因为在执行BGSAVE命令或BGREWRITEAOF命令的过程中，Redis需要创建当前服务器进程的子进程，而大多数操作系统都采用写时复制( copy-on-write)技术来优化子进程的使用效率，所以在子进程存在期间，服务器会提高执行扩展操作所需的负载因子，从而尽可能地避免在子进程存在期间进行哈希表扩展操作，这可以避免不必要的内存写入操作,最大限度地节约内存。

另一方面，当哈希表的负载因子小于0.1时，程序自动开始对哈希表执行收缩操作。

扩展和收缩哈希表的工作可以通过执行rehash（重新散列）操作来完成，Redis对字典的哈希表执行rehash的步骤如下:

1.  为字典的ht [1]哈希表分配空间，这个哈希表的空间大小取决于要执行的操作，以及ht[0]当前包含的键值对数量（也即是ht[0].used属性的值):

-   -   如果执行的是扩展操作，那么ht [1]的大小为**第一个大于等于**ht[0].used*2的2"( 2的n次方幂);
    -   如果执行的是收缩操作，那么ht [1]的大小为第一个大于等于ht[0].used的2"

2.  将保存在ht[0]中的所有键值对rehash到ht[1]上面： rehash 指的是重新计算键的哈希值和索引值，然后将键值对放置到ht[1]哈希表的指定位置上。
2.  当ht[0]包含的所有键值对都迁移到了ht [1]之后(ht[0]变为空表)，释放ht[0]，将ht[1]设置为ht[0]，并在ht[1]新创建一个空白哈希表，为下一次rehash做准备。

### 渐进式rehash

rehash并不是一次完成的，否则当key过多的时候，计算所需的时间足以让redis服务器瘫痪。因此，为了避免 rehash对服务器性能造成影响，服务器不是一次性将ht[0]里面的所有键值对全部rehash到ht[1]，而是分多次、渐进式地将ht[0]里面的键值对慢慢地rehash到ht[1]。

以下是哈希表渐进式rehash的详细步骤:

1.  为ht[1]分配空间，让字典同时持有ht[0]和 ht[1]两个哈希表。
1.  在字典中维持一个索引计数器变量rehashidx，并将它的值设置为0，表示rehash工作正式开始。

<!---->

3.  在rehash进行期间，每次对字典执行添加、删除、查找或者更新操作时，程序除了执行指定的操作以外，还会顺带将ht[0]哈希表在rehashidx索引上的所有键值对rehash到ht[1]，当rehash 工作完成之后，程序将rehashidx属性的值增一。
3.  随着字典操作的不断执行，最终在某个时间点上，ht[0]的所有键值对都会被rehash至 ht [1]，这时程序将rehashidx属性的值设为-1，表示rehash操作已完成。

渐进式rehash 的好处在于它采取分而治之的方式，将rehash键值对所需的计算工作均摊到对字典的每个添加、删除、查找和更新操作上，从而避免了集中式rehash而带来的庞大计算量。

### 渐进式rehash执行期间的哈希表操作

因为在进行渐进式rehash的过程中，字典会同时使用ht[0]和ht [1]两个哈希表，所以在渐进式rehash进行期间，字典的删除( delete)、查找( find)、更新(update）等操作会在两个哈希表上进行。例如，**要在字典里面查找一个键的话，程序会先在ht[0]里面进行查找，如果没找到的话，就会继续到ht[1]里面进行查找，诸如此类。**

另外，**在渐进式rehash执行期间，新添加到字典的键值对一律会被保存到ht[1]里面，而ht[0]则不再进行任何添加操作，这一措施保证了ht[0]包含的键值对数量会只减不增，并随着rehash操作的执行而最终变成空表。**
# 跳表

跳跃表（ skiplist）是一种有序数据结构，它通过在每个节点中维持多个指向其他节点的指针,从而达到快速访问节点的目的。

跳跃表支持平均O(logN)、最坏O(N)复杂度的节点查找，还可以通过顺序性操作来批量处理节点。

在大部分情况下，跳跃表的效率可以和平衡树相媲美，并且因为跳跃表的实现比平衡树要来得更为简单，所以有不少程序都使用跳跃表来代替平衡树。

Redis使用跳跃表作为有序集合键的底层实现之一，**如果一个有序集合包含的元素数量比较多，又或者有序集合中元素的成员（member)是比较长的字符串时，Redis就会使用跳跃表来作为有序集合键的底层实现(ordered_set)。**

和链表、字典等数据结构被广泛地应用在Redis内部不同，Redis只在两个地方用到了跳跃表，一个是实现有序集合键，另一个是在集群节点中用作内部数据结构。

## 定义

Redis 的跳跃表由redis.h/zskiplistNode和redis.h/zskiplist两个结构定义，其中zskiplistNode结构用于表示跳跃表节点，而zskiplist结构则用于保存跳跃表节点的相关信息，比如节点的数量，以及指向表头节点和表尾节点的指针等等。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3e83c9ba2da4660a22d702640029f40~tplv-k3u1fbpfcp-zoom-1.image)

图5-1展示了一个跳跃表示例，位于图片最左边的是zskiplist结构，该结构包含以下属性:

-   header: 指向跳跃表的表头节点。
-   tail: 指向跳跃表的表尾节点。

<!---->

-   level: 记录目前跳跃表内，层数最大的那个节点的层数 **（表头节点的层数不计算在内)。**
-   length: 记录跳跃表的长度，也即是，跳跃表目前包含节点的数量 **（表头节点不计算在内)**

位于zskiplist结构右方的是四个zskiplistNode结构，该结构包含以下属性:

-   **层( level ): ** 节点中用L1、L2、L3等字样标记节点的各个层，L1代表第一层，L2代表第二层，以此类推。每个层都带有两个属性: **前进指针和跨度**。前进指针用于访问位于表尾方向的其他节点，而跨度则记录了前进指针所指向节点和当前节点的距离。在上面的图片中，连线上带有数字的箭头就代表前进指针，而那个数字就是跨度。当程序从表头向表尾进行遍历时，访问会沿着层的前进指针进行。
-   **后退（ backward）指针:** 节点中用Bw字样标记节点的后退指针，它指向位于当前节点的前一个节点。后退指针在程序从表尾向表头遍历时使用。表头节点没有BW指针

<!---->

-   **分值( score ): ** 各个节点中的1.0、2.0和3.0是节点所保存的分值。在跳跃表中，节点按各自所保存的分值从小到大排列。
-   **成员对象（obj):** 各个节点中的o1、o2和 o3是节点所保存的成员对象。

注意表头节点和其他节点的构造是一样的:表头节点也有后退指针、分值和成员对象，不过表头节点的这些属性都不会被用到，所以图中省略了这些部分，只显示了表头节点的各个层。

### 跳跃表节点

跳跃表节点的实现由redis.h/zskiplistNode结构定义:

```
typedef struct zskiplistNode {
    
    //层
    struct zskiplistLevel {
        //前进指针
        struct zskiplistNode *forward;
        //跨度
        unsigned int span;
    }level[];
    
	//后退指针
	struct zskiplistNode *backward;
    
    //分值
	double score;
    
   //成员对象
   robj *obj; //注：这里是一个字符串对象，之所以用robj可能是字符串对象有三种实现方式
    
}zskiplistNode;
```

#### 层

跳跃表节点的level数组可以包含多个元素，每个元素都包含一个指向其他节点的指针，程序可以通过这些层来加快访问其他节点的速度，一般来说，层的数量越多，访问其他节点的速度就越快。

每次创建一个新跳跃表节点的时候，程序都根据幂次定律(power law，越大的数出现的概率越小）随机生成一个介于1和32之间的值作为level数组的大小，这个大小就是层的“高度”。

#### 前进指针

每个层都有一个指向表尾方向的前进指针( level[i].forward属性)，用于从表头向表尾方向访问节点。

#### 跨度

层的跨度( level[i].span属性）用于记录两个节点之间的距离: 两个节点之间的跨度越大，它们相距得就越远。指向NULL 的所有前进指针的跨度都为0，因为它们没有连向任何节点。

遍历操作只使用前进指针就可以完成，跨度实际上是用来计算排位（rank )的:在查找某个节点的过程中，将沿途访问过的所有层的跨度累计起来，得到的结果就是目标节点在跳跃表中的排位。

#### 后退指针

节点的后退指针（ backward属性）用于从表尾向表头方向访问节点:跟可以一次跳过多个节点的前进指针不同，因为每个节点只有一个后退指针，所以每次只能后退至前一个节点。

#### 分值和成员

节点的分值( score属性）是一个double类型的浮点数，跳跃表中的所有节点都按分值从小到大来排序。

节点的成员对象（ obj属性）是一个指针，它指向一个字符串对象，而字符串对象则保存着一个SDS值。

在同一个跳跃表中，各个节点保存的成员对象必须是唯一的，但是多个节点保存的分值却可以是相同的:分值相同的节点将按照成员对象在字典序中的大小来进行排序，成员对象较小的节点会排在前面。

### 跳跃表

zskiplist结构的定义如下:

```
typedef struct zskiplist {
    
    //表头节点和表尾节点
    structz skiplistNode *header, *tail;
    
    //表中节点的数量
    unsigned long length;
    
    //表中层数最大的节点的层数
    int level;
    
} zskiplist;
```

header和tail指针分别指向跳跃表的表头和表尾节点，通过这两个指针，程序定位表头节点和表尾节点的复杂度为O(1)。

通过使用length属性来记录节点的数量，程序可以在O(1)复杂度内返回跳跃表的长度。level属性则用于在O(1)复杂度内获取跳跃表中层高最大的那个节点的层数量，注意表头节点的层高并不计算在内。

# 整数集合

整数集合( intset）是集合键的底层实现之一，当一个集合只包含整数值元素，并且这个集合的元素数量不多时，Redis就会使用整数集合作为集合键的底层实现。

## 定义

整数集合( intset）是Redis用于保存整数值的集合抽象数据结构，它可以保存类型为int16_t、int32_t或者int64_t的整数值，并且保证集合中不会出现重复元素。

每个intset.h/intset 结构表示一个整数集合:

```
typedef struct intset {
    
	//编码方式
	uint32_t encoding;
    
    //集合包含的元素数量
    uint32_t length;
    
    //保存元素的数组
	int8_t contents[];
    
}intset;
```

contents数组是整数集合的底层实现:整数集合的每个元素都是contents数组的一个数组项( item )，**各个项在数组中按值的大小从小到大有序地排列**，并且数组中不包含任何重复项。

length属性记录了整数集合包含的元素数量，也即是contents数组的长度。

虽然intset结构将contents属性声明为int8 t类型的数组，但实际上 contents数组并不保存任何int8_t类型的值，contents数组的真正类型取决于encoding属性的值:

如果encoding属性的值为工NTSET_ENC_INT16，那么contents就是一个int16_t类型的数组，数组里的每个项都是一个int16_t类型的整数值(最小值为-32768，最大值为32767 )。以此类推

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41130518bfa548c0953586df69949b65~tplv-k3u1fbpfcp-zoom-1.image)

## 升级

每当我们要将一个新元素添加到整数集合里面，并且新元素的类型比整数集合现有所有元素的类型都要长时，整数集合需要先进行升级(upgrade)，然后才能将新元素添加到整数集合里面。

升级整数集合并添加新元素共分为三步进行:

1.  根据新元素的类型，扩展整数集合底层数组的空间大小，并为新元素分配空间。
1.  将底层数组现有的所有元素都转换成与新元素相同的类型，并将类型转换后的元素放置到正确的位上，而且在放置元素的过程中，需要继续维持底层数组的有序性质不变。

<!---->

3.  将新元素添加到底层数组里面。

举个例子：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34e7e4b85db549f4be644d132009c0dc~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc392018d0fc42b2823430928a9aedfc~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87f31933d0974a4dafe0f3a65f379c43~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ded9b9ced9d74177b5bf05a63b2dc98d~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/402a8d463cce49eeba7951331e46f652~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52a7b35fa74c46fcbf2f522114cfc359~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67e96e0aae5b46cda0755ec5404f4ba6~tplv-k3u1fbpfcp-zoom-1.image)


因为引发升级的新元素的长度总是比整数集合现有所有元素的长度都大，所以这个新元素的值要么就大于所有现有元素，要么就小于所有现有元素:

-   在新元素小于所有现有元素的情况下，新元素会被放置在底层数组的最开头(索引0 );

<!---->

-   在新元素大于所有现有元素的情况下，新元素会被放置在底层数组的最末尾(索引 length-1 )。

**升级有两个好处：**

1.  提升灵活性，因为C语言是静态类型语言，为了避免类型错误，我们通常不会将两种不同类型的值放在同一个数据结构里面。而整数集合会自动升级来适应新元素，所以我们不用担心
1.  节约内存，整数集合现在的做法既可以让集合能同时保存三种不同类型的值，又可以确保升级操三只会在有需要的时候进行,这可以尽量节省内存。

**注：整数集合不支持降级**

# 压缩列表

压缩列表(ziplist)是列表键和哈希键的底层实现之一。**当一个列表键只包含少量列表项，并且每个列表项要么就是小整数值，要么就是长度比较短的字符串，那么Redis就会使用压缩列表来做列表键的底层实现。**

## 定义

压缩列表是Redis为了节约内存而开发的，是由一系列特殊编码的连续内存块组成的顺序型( sequential)数据结构。一个压缩列表可以包含任意多个节点( entry)，每个节点可以保存一个字节数组或者一个整数值。

图7-1展示了压缩列表的各个组成部分，表7-1则记录了各个组成部分的类型、长度以及用途

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/032d99bfd7a646708b93fb4b3b60fe58~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b31e7984f794d82a424d584c2ada65e~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5eb767f1cb3641a88286b0227b964d09~tplv-k3u1fbpfcp-zoom-1.image)

### 压缩列表节点

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4ec0d46844c43c0a0e042d7a1391841~tplv-k3u1fbpfcp-zoom-1.image)

#### previous_entry_length

节点的previous_entry_length属性以字节为单位，记录了压缩列表中前一个节点的长度。 previous_entry_length属性的长度可以是1字节或者5字节:

-   如果前一节点的长度小于254字节，那么previous_entry_length属性的长度为1字节:前一节点的长度就保存在这一个字节里面。
-   如果前一节点的长度大于等于254字节，那么previous_entry_length属性的长度为5字节:其中属性的第一字节会被设置为0xFE(十进制值254 )，而之后的四个字节则用于保存前一节点的长度。

例：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a19bca42877448ffbf749b7d38299223~tplv-k3u1fbpfcp-zoom-1.image)

因为节点的previous_entry_length属性记录了前一个节点的长度，所以程序可以通过指针运算，根据当前节点的起始地址来计算出前一个节点的起始地址。压缩列表的从表尾向表头遍历操作就是使用这一原理实现的，只要我们拥有了一个指向某个节点起始地址的指针，那么通过这个指针以及这个节点的previous_entry_length属性，程序就可以一直向前一个节点回溯，最终到达压缩列表的表头节点。

#### encoding

节点的encoding属性记录了节点的content属性所保存数据的类型以及长度:

-   一字节、两字节或者五字节长，值的最高位为00、01或者10的是字节数组编码：这种编码表示节点的content属性保存着字节数组，数组的长度由编码除去最高两位之后的其他位记录;
-   一字节长，值的最高位以11开头的是整数编码：这种编码表示节点的content属性保存着整数值，整数值的类型和长度由编码除去最高两位之后的其他位记录;

#### content

节点的content属性负责保存节点的值，节点值可以是一个字节数组或者整数，值的类型和长度由节点的encoding属性决定。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59d2da5e1c8b4a23a1128a25007d31df~tplv-k3u1fbpfcp-zoom-1.image)

## 连锁更新

某种情况下，添加新节点会导致所有节点都要进行扩展：比如一个列表，所有节点都是253字节，然后插入一个新节点，长度是254字节，那么新节点的后一节点的previous_entry_key就必须增加一个字节，总长度也变成254字节，以此类推，后面所有节点都需要重新分配空间。

删除某个节点也会导致连锁更新，如：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f56053c44424b379a0389ae9e1e0f5c~tplv-k3u1fbpfcp-zoom-1.image)

e1必须扩容才能记录small前一个节点big的大小，同样也会导致后续所有节点扩容

因为连锁更新在最坏情况下需要对压缩列表执行N次空间重分配操作，而每次空间重分配的最坏复杂度为

O(N)，所以连锁更新的最坏复杂度为O(N*)。

要注意的是，尽管连锁更新的复杂度较高，但它真正造成性能问题的几率是很低的：首先，压缩列表里要恰好有多个连续的、长度介于250字节至253字节之间的节点，连锁更新才有可能被引发，在实际中，这种情况并不多见；其次，即使出现连锁更新，但只要被更新的节点数量不多，就不会对性能造成任何影响。

# 对象

在前面的数个章节里，我们陆续介绍了Redis用到的所有主要数据结构，比如简单动态字符串( SDS)、双端链表、字典、压缩列表、整数集合等等。

Redis并没有直接使用这些数据结构来实现键值对数据库，而是基于这些数据结构创建了一个对象系统，这个系统包含字符串对象、列表对象、哈希对象、集合对象和有序集合对象这五种类型的对象，每种对象都用到了至少一种我们前面所介绍的数据结构。

通过这五种不同类型的对象，Redis可以在执行命令之前，根据对象的类型来判断一个对象是否可以执行给定的命令。使用对象的另一个好处是，我们可以针对不同的使用场景,为对象设置多种不同的数据结构实现，从而优化对象在不同场景下的使用效率。

除此之外，Redis的对象系统还实现了基于引用计数技术的内存回收机制，当程序不再使用某个对象的时候，这个对象所占用的内存就会被自动释放;另外，Redis还通过引用计数技术实现了对象共享机制，这一机制可以在适当的条件下，通过让多个数据库键共享同-个对象来节约内存。

最后，Redis 的对象带有访问时间记录信息，该信息可以用于计算数据库键的空转时长，在服务器启用了maxmemory功能的情况下，空转时长较大的那些键可能会优先被服务器删除。

## 对象的类型和编码

Redis使用对象来表示数据库中的键和值，每次当我们在Redis 的数据库中新创建一个键值对时，我们至少会创建两个对象，一个对象用作键值对的键（键对象)，另一个对象用作键值对的值（值对象)。

举个例子，`set msg hello world`在数据库中创建了一个新的键值对，其中键值对的键是一个包含了字符串值"msg"的对象，而键值对的值则是一个包含了字符串值"hello world"的对象:

Redis 中的每个对象都由一个redisobject结构表示，该结构中和保存数据有关的三个属性分别是type属性、encoding属性和 ptr属性:

```
typedef struct redisobject {
    //类型
    unsigned type : 4; //表示占4位
    
    //编码
    unsigned encoding : 4;
    
    //指向底层实现数据结构的指针
    void *ptr;
    
    //...
} robj;
```

### 类型

对象的type属性记录了对象的类型，这个属性的值可以是表8-1列出的常量的其中一个。

对于Redis数据库保存的键值对来说,**键总是一个字符串对象，而值则可以是字符串对象、列表对象、哈希对象、集合对象或者有序集合对象的其中一种**，因此:

-   当我们称呼一个数据库键为“字符串键”时，我们指的是“这个数据库键所对应的值为字符串对象”;
-   当我们称呼一个键为“列表键”时，我们指的是“这个数据库键所对应的值为列表对象”。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2917686cb70f4083b67b51df94eafc73~tplv-k3u1fbpfcp-zoom-1.image)

TYPE命令的实现方式也与此类似，当我们对一个数据库键执行TYPE命令时，命令返回的结果为数据库键对应的值对象的类型，而不是键对象的类型:

#### 编码和底层实现

对象的ptr指针指向对象的底层实现数据结构，而这些数据结构由对象的encoding属性决定。

encoding属性记录了对象所使用的编码，也即是说这个对象使用了什么数据结构作为对象的底层实现，这个属性的值可以是表8-3列出的常量的其中一个。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bbf28f2904c74c2d958c5ea2dc0f6e2d~tplv-k3u1fbpfcp-zoom-1.image)

每种类型的对象都至少使用了两种不同的编码，表8-4列出了每种类型的对象可以使用的编码。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8801aa1df815463f93c03fb3bc9b100a~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/316f6c3517ad48fbbdcaf633defcdbaa~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d1e74d911bb4c7a9ed6c2b8f3789476~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2927bf2be1b447ce857627af97c9ccc3~tplv-k3u1fbpfcp-zoom-1.image)

通过encoding属性来设定对象所使用的编码，而不是为特定类型的对象关联一种固定的编码，极大地提升了Redis的灵活性和效率，因为Redis可以根据不同的使用场景来为一个对象设置不同的编码，从而优化对象在某一场景下的效率。

举个例子，在列表对象包含的元素比较少时，Redis使用压缩列表作为列表对象的底层实现:因为压缩列表比双端链表更节约内存，并且在元素数量较少时，在内存中以连续块方式保存的压缩列表比起双端链表可以更快被载入到缓存中；随着列表对象包含的元素越来越多，使用压缩列表来保存元素的优势逐渐消失时，对象就会将底层实现从压缩列表转向功能更强、也更适合保存大量元素的双端链表上面；其他类型的对象也会通过使用多种不同的编码来进行类似的优化。

## 字符串对象

字符串对象的编码可以是int、raw或者embstr。

如果一个字符串对象保存的是整数值，并且这个整数值可以用long类型来表示，那么字符串对象会将整数值保存在字符串对象结构的ptr属性里面（将void*转换成long )，并将字符串对象的编码设置为int。

如果字符串对象保存的是一个字符串值，并且这个字符串值的长度大于32字节，那么字符串对象将使用一个简单动态字符串(SDS）来保存这个字符串值，并将对象的编码设置为raw。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1e22f68bd9342f18c5a7b39186122b5~tplv-k3u1fbpfcp-zoom-1.image)

如果字符串对象保存的是一个字符串值，并且这个字符串值的长度小于等于32字节，那么字符串对象将使用embstr编码的方式来保存这个字符串值。

embstr编码是专门用于保存短字符串的一种优化编码方式，这种编码和raw编码一样，都使用redisobject结构和sdshdr结构来表示字符串对象，但raw编码会调用两次内存分配函数来分别创建redisobject结构和sdshdr结构，而embstr编码则通过**调用一次内存分配函数来分配一块连续的空间**，空间中依次包含redisobject和 sdshdr两个结构，如图8-3所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/545bf837b2304e5eabfa6d63c801ed35~tplv-k3u1fbpfcp-zoom-1.image)

**embstr的好处：**

-   embstr编码将创建字符串对象所需的内存分配次数从raw编码的两次降低为一次。
-   释放embstr编码的字符串对象只需要调用一次内存释放函数，而释放raw编码的字符串对象需要调用两次内存释放函数。

<!---->

-   因为embstr编码的字符串对象的所有数据都保存在一块连续的内存里面，所以这种编码的字符串对象比起raw编码的字符串对象能够更好地利用缓存带来的优势。

最后要说的是，可以用long double类型表示的浮点数在Redis中也是作为字符串直来保存的。如果我们要保存一个浮点数到字符串对象里面，那么程序会先将这个浮点数转奂成字符串值,然后再保存转换所得的字符串值。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e00964dc2c554bb2847cf455f5cc1385~tplv-k3u1fbpfcp-zoom-1.image)

在有需要的时候，程序会将保存在字符串对象里面的字符串值转换回浮点数值，执行某些操作，然后再将执行操作所得的浮点数值转换回字符串值，并继续保存在字符串对象里面。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14754b89804a49c1bf9f44707017d015~tplv-k3u1fbpfcp-zoom-1.image)

那么程序首先会取出字符串对象里面保存的字符串值"3.14"，将它转换回浮点数值3.14，然后把3.14和2.0相加得出的值5.14转换成字符串"5.14"，并将这个"5.14"保存到字符串对象里面。表8-6总结并列出了字符串对象保存各种不同类型的值所使用的编码方式。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10cea233d11746cb971a2246e2642288~tplv-k3u1fbpfcp-zoom-1.image)

### 编码的转换

int编码的字符串对象和embstr编码的字符串对象在条件满足的情况下，会被转换为raw编码的字符串对象。

对于int编码的字符串对象来说，如果我们向对象执行了一些命令，使得这个对象保存的不再是整数值，而是一个字符串值，那么字符串对象的编码将从int变为raw。

```
redis> SET number 10086
OK

redis> OBJECT ENCODING number
"int"

redis> APPEND number " is a good number ! "
(integer)23
    
redis> GET number
"10086 is a good number ! "

redis> OBJECT ENCODING number
"raw"
```

另外，因为**Redis没有为embstr编码的字符串对象编写任何相应的修改程序（只有int编码的字符串对象和raw编码的字符串对象有这些程序)，所以embstr编码的字符串对象实际上是只读的。** 当我们对embstr编码的字符串对象执行任何修改命令时，**程序会先将对象的编码从embstr转换成raw，然后再执行修改命令。因为这个原因，embstr编码的字符串对象在执行修改命令之后，总会变成一个raw编码的字符串对象。**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b46ec6c6b29457fa61671c7eef4c2e9~tplv-k3u1fbpfcp-zoom-1.image)

### 字符串命令的实现

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6e182d676344c17863fe6d0435f21d5~tplv-k3u1fbpfcp-zoom-1.image)

## 列表对象

\


列表对象的编码可以是ziplist或者linkedlist。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea9b1d4238a641aaae3cc0835241dcce~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bae9956906df4b07b160e56730260770~tplv-k3u1fbpfcp-zoom-1.image)

注意，linkedlist编码的列表对象在底层的双端链表结构中包含了多个字符串对象，这种嵌套字符串对象的行为在稍后介绍的哈希对象、集合对象和有序集合对象中都会出现，字符串对象是Redis五种类型的对象中唯一一种会被其他四种类型对象嵌套的对象。**即，链表的value类型是一个SDS**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ab94086b5d34b5aa59aab2fdd7e71f4~tplv-k3u1fbpfcp-zoom-1.image)

感觉上面这个图也不对，应该是指向一个链表结构，链表的值是SDS，同时注意链表是双向无环的

### 编码转换

当列表对象可以同时满足以下两个条件时，列表对象使用ziplist编码:

-   列表对象保存的所有字符串元素的长度都小于64字节;
-   列表对象保存的元素数量小于512个；

不能满足这两个条件的列表对象需要使用linkedlist编码。

注：以上两个条件的上限值是可以修改的，具体请看配置文件中关于list-max-ziplist-value选项和list-max-ziplist-entries 选项的说明。

### 命令的实现

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/155aeea827864b619785193b5c77c9fd~tplv-k3u1fbpfcp-zoom-1.image)

## 哈希对象

哈希对象的编码可以是ziplist或者hashtable。

ziplist编码的哈希对象使用压缩列表作为底层实现，每当有新的键值对(field-value)要加入到哈希对象时，程序会先将保存了键的压缩列表节点推入到压缩列表表尾，然后再将保存了值的压缩列表节点推人到压缩列表表尾。因此:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d80bad5a7df4ecda9d632297f7d0325~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18333b0b1e434aa3816ac3f7d4e45dd8~tplv-k3u1fbpfcp-zoom-1.image)

注意，与使用篇`hset key field value`对比，这里的profile对象是key，而name、age、career是field

另一方面，hashtable编码的哈希对象使用字典作为底层实现，哈希对象中的每个键值对都使用一个字典键值对来保存:

-   字典的每个键都是一个字符串对象，对象中保存了键值对的键;
-   字典的每个值都是一个字符串对象，对象中保存了键值对的值。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ee28b30db0746cbb94a2fb00759926a~tplv-k3u1fbpfcp-zoom-1.image)

### 编码转换

当哈希对象可以同时满足以下两个条件时，哈希对象使用ziplist编码:

-   哈希对象保存的所有键值对的键和值的字符串长度都小于64字节;
-   哈希对象保存的键值对数量小于512个;

不能满足这两个条件的哈希对象需要使用hashtable编码。

注：这两个条件的上限值是可以修改的，具体请看配置文件中关于hash-max-ziplist-value选项和hash-max-ziplist-entries选项的说明。

### 命令实现

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/665c920c253f44f6aa69f72d02259a6f~tplv-k3u1fbpfcp-zoom-1.image)

## 集合对象

集合对象的编码可以是intset或者hashtable。

intset编码的集合对象使用整数集合作为底层实现，集合对象包含的所有元素都被保存在整数集合里面。

另一方面，hashtable编码的集合对象使用字典作为底层实现，字典的每个键都是一个字符串对象，每个字符串对象包含了一个集合元素，而字典的值则全部被设置为NULL。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01fffcd5bac846b4a19e04d3a6b22d01~tplv-k3u1fbpfcp-zoom-1.image)

### 转换

当集合对象可以同时满足以下两个条件时，对象使用intset 编码:

-   集合对象保存的所有元素都是整数值;
-   集合对象保存的元素数量不超过512个。

不能满足这两个条件的集合对象需要使用hashtable编码。

注：第二个条件的上限值是可以修改的，具体请看配置文件中关于set-max-intset-entries选项的说明。

### 命令实现

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83c60b5a7e344ce392ab71ed38a913aa~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a4076257c9641758a6d57c9029c2d40~tplv-k3u1fbpfcp-zoom-1.image)

## ordered_set

有序集合的编码可以是ziplist或者skiplist。

ziplist编码的压缩列表对象使用压缩列表作为底层实现，每个集合元素使用两个紧挨在一起的压缩列表节点来保存，第一个节点保存元素的成员( member )，而第二个元素则保存元素的分值( score )。

压缩列表内的集合元素按分值从小到大进行排序，分值较小的元素被放置在靠近表头的方向,而分值较大的元素则被放置在靠近表尾的方向。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60d897a6a2264b6083feea8d6045197e~tplv-k3u1fbpfcp-zoom-1.image)


我们来看看用skiplist存储内容的有序集合：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ab89a7afa6f4aba85c42587a3404e75~tplv-k3u1fbpfcp-zoom-1.image)

zset结构中的zsl跳跃表按分值从小到大保存了所有集合元素，每个跳跃表节点都保存了一个集合元素：跳跃表节点的object属性保存了元素的成员(也就是值)，而跳跃表节点的score属性则保存了元素的分值。**通过这个跳跃表，程序可以对有序集合进行范围型操作，比如ZRANK、ZRANGE等命令就是基于跳跃表API来实现的。**

除此之外，zset结构中的dict字典为有序集合创建了一个从成员到分值的映射，字典中的每个键值对都保存了一个集合元素:字典的键保存了元素的成员，而字典的值则保存了元素的分值。**通过这个字典，程序可以用O(1)复杂度查找给定成员的分值，ZSCORE命令就是根据这一特性实现的**，而很多其他有序集合命令都在实现的内部用到了这一特性。

有序集合每个元素的成员都是一个字符串对象，而每个元素的分值都是一个double类型的浮点数。值得一提的是，**虽然zset结构同时使用跳跃表和字典来保存有序集合元素,但这两种数据结构都会通过指针来共享相同元素的成员和分值，所以同时使用跳跃表和字典来保存集合元素不会产生任何重复成员或者分值，也不会因此而浪费额外的内存。**

**也就是说，同时使用字典和跳表是为了最小时间复杂度查询元素和元素的分值**

### 编码的转换

当有序集合对象可以同时满足以下两个条件时，对象使用ziplist编码;

-   有序集合保存的元素数量小于128个;
-   有序集合保存的所有元素成员的长度都小于64字节;

不能满足以上两个条件的有序集合对象将使用skiplist编码。

以上两个条件的上限值是可以修改的，具体请看配置文件中关于zset-max-ziplist-entries选项和zset-max-ziplist-value 选项的说明。

### 命令的实现

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b82ecc5cf7249868c85b8148b59c54a~tplv-k3u1fbpfcp-zoom-1.image)

## 命令检查与多态

### 命令检查

Redis中用于操作键的命令基本上可以分为两种类型。

-   -   其中一种命令可以对任何类型的键执行，比如说 DEL命令、EXPIRE命令、RENAME命令、TYPE命令、OBJECT命令等。
    -   而另一种命令只能对特定类型的键执行，比如说: SET、GET、APPEND、STRLEN等命令只能对字符串键执行

为了确保只有指定类型的键可以执行某些特定的命令，在执行一个类型特定的命令之前，Redis 会先检查输入键的类型是否正确，然后再决定是否执行给定的命令。类型特定命令所进行的类型检查是通过redisobject结构的type属性来实现的:

-   在执行一个类型特定命令之前，服务器会先检查输人数据库键的值对象是否为执行命令所需的类型，如果是的话，服务器就对键执行指定的命令;
-   否则，服务器将拒绝执行命令，并向客户端返回一个类型错误。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08dfdc25ac974f72af682ec0ec0fcabd~tplv-k3u1fbpfcp-zoom-1.image)

### 多态

Redis除了会根据值对象的类型来判断键是否能够执行指定命令之外，还会根据值对象的编码方式，选择正确的命令实现代码来执行命令。

举个例子，在前面介绍列表对象的编码时我们说过，列表对象有ziplist和linkedlist两种编码可用，其中前者使用压缩列表API来实现列表命令，而后者则使用双端链表API来实现列表命令。当我们执行`LLEN`命令，那么服务器就需要根据键的值的编码来选择正确的`LLEN`实现

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2d68c6f38bf4159a607628b1d449853~tplv-k3u1fbpfcp-zoom-1.image)

## 内存回收

因为C语言并不具备自动内存回收功能，所以Redis在自己的对象系统中构建了一个引用计数( reference counting)技术实现的内存回收机制，通过这一机制，程序可以通过跟踪对象的引用计数信息，在适当的时候自动释放对象并进行内存回收。

每个对象的引用计数信息由redisobject结构的refcount属性记录:

```
typedef struct redisObject {
    
	//...
    
	//引用计数
    int refcount;
    
	// ...
) robj;
```

对象的引用计数信息会随着对象的使用状态而不断变化:

-   在创建一个新对象时，引用计数的值会被初始化为1;
-   当对象被一个新程序使用时，它的引用计数值会被增一;

<!---->

-   当对象不再被一个程序使用时，它的引用计数值会被减一;
-   当对象的引用计数值变为0时，对象所占用的内存会被释放。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74176cbb99f24ad4945f57d22643a8d1~tplv-k3u1fbpfcp-zoom-1.image)

## 对象共享

除了用于实现引用计数内存回收机制之外，对象的引用计数属性还带有对象共享的作用。举个例子，假设键A创建了一个包含整数值100的字符串对象作为值对象，如图8-20所示。

如果这时键B也要创建一个同样保存了整数值100的字符串对象作为值对象，那么服务器有以下两种做法:

1.  为键B新创建一个包含整数值100的字符串对象;
1.  让键A和键B共享同一个字符串对象;

以上两种方法很明显是第二种方法更节约内存。

在Redis 中，让多个键共享同一个值对象需要执行以下两个步骤:

1.  将数据库键的值指针指向一个现有的值对象;
1.  将被共享的值对象的引用计数增一。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/243d0948a2fc4fe590880d8c0cd0e5c3~tplv-k3u1fbpfcp-zoom-1.image)

\


目前来说，Redis 会在初始化服务器时，创建一万个字符串对象，这些对象包含了从0到9999的所有整数值，当服务器需要用到值为О到9999的字符串对象时，服务器就会使用这些共享对象，而不是新创建对象。

创建共享字符串对象的数量可以通过修改redis.h/REDIS_SHARED_INTEGERS常量来修改。

举个例子，如果我们创建一个值为100的键A，并使用OBJECT REFCOUNT命令查看键A的值对象的引用计数，我们会发现值对象的引用计数为2:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07ace3c827ac4d96adc06720b7d39e48~tplv-k3u1fbpfcp-zoom-1.image)

另外，这些共享对象不单单只有字符串键可以使用，那些在数据结构中嵌套了字符串对象的对象( linkedlist编码的列表对象、hashtable编码的哈希对象、hashtable编码的集合对象，以及zset编码的有序集合对象）都可以使用这些共享对象。

## 对象的空转时长

除了前面介绍过的type、encoding、ptr和refcount 四个属性之外，redisobject结构包含的最后一个属性为lru属性，该属性记录了对象最后一次被命令程序访问的时间:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c0b92ac4c33427f8d9b4e93ed01cb07~tplv-k3u1fbpfcp-zoom-1.image)

`OBJECT IDLETIME`命令可以打印出给定键的空转时长，这一空转时长就是通过将当前时间减去键的值对象的lru时间计算得出的:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05ddb10d82494c17ae2f154b0e22fc09~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f8aa8864edf4082a505ec413e3fc4e0~tplv-k3u1fbpfcp-zoom-1.image)

注：`OBJECT IDLETIME`命令不会修改lru属性

**这个属性被用在当内存满时逐出对象的选择上。**

## 总结

-   Redis数据库中的每个键值对的键和值都是一个对象。
-   Redis 共有字符串、列表、哈希、集合、有序集合五种类型的对象，每种类型的对象至少都有两种或以上的编码方式，不同的编码可以在不同的使用场景上优化对象的使用效率。

<!---->

-   服务器在执行某些命令之前，会先检查给定键的类型能否执行指定的命令，而检查一个键的类型就是检查键的值对象的类型。
-   Redis 的对象系统带有引用计数实现的内存回收机制，当一个对象不再被使用时，该对象所占用的内存就会被自动释放。

<!---->

-   Redis会共享值为0到9999的字符串对象。
-   对象会记录自己的最后一次被访问的时间，这个时间可以用于计算对象的空转时间。

