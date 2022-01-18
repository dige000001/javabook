## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# JVM内存模型

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d815393074f4468ac534f83c559eff4~tplv-k3u1fbpfcp-zoom-1.image)

## 虚拟机栈：

为了**保证线程中的局部变量不被其他线程访问到**，**虚拟机栈和本地方法栈是线程私有的。**

每个 Java 方法在执行的同时会创建一个栈帧**用于存储局部变量表、操作数栈、常量池引用等**信息。从方法调用直至执行完成的过程，就对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。


-   栈帧是用于支持JVM进行方法调用和方法执行的数据结构
-   栈帧随着方法调用而创建，随着方法结束而销毁

<!---->

-   栈帧里面存储了方法的局部变量、操作数栈、动态连接、方法返回地址等信息

**这么一看，每个方法都有一个栈帧哈**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21d3d49dd3f14300a374aa798ee73f80~tplv-k3u1fbpfcp-zoom-1.image)

### 局部变量表

局部变量表用于存放方法参数和方法内部定义的局部变量 ，存放了编译期可知的各种Java虚拟机基本数据类型（boolean、byte、char、short、int、 float、long、double）、对象引用（reference类型，它并不等同于对象本身，可能是一个指向对象起始 地址的引用指针，也可能是指向一个代表对象的句柄或者其他与此对象相关的位置）和returnAddress 类型（指向了一条字节码指令的地址）。 

在《Java虚拟机规范》中，对这个内存区域规定了两类异常状况：如果线程请求的栈深度大于虚 拟机所允许的深度，将抛出StackOverflowError异常；如果Java虚拟机栈容量可以动态扩展，当栈扩 展时无法申请到足够的内存会抛出OutOfMemoryError异常。 HotSpot虚拟机的栈容量是不可以动态扩展的，以前的Classic虚拟机倒是可以。所以在HotSpot虚拟 机上是不会由于虚拟机栈无法扩展而导致OutOfMemoryError异常——只要线程申请栈空间成功了就不 会有OOM，但是如果申请时就失败，仍然是会出现OOM异常的 

1.  **以变量槽slot为单位，目前一个slot存放32位以内的数据类型**
1.  **对于64位的数据占2个slot**

<!---->

3.  **对于** **实例方法，** **第0位slot存放的是this，然后从1到n ,依次分配给参数列表**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f0d8a889b5c4d0d9ce39c39b74b6aba~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d617851b50240a988e85be46c60e9db~tplv-k3u1fbpfcp-zoom-1.image)




而如果设成静态方法

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa4c9da4b1734049a0cf5f2acc36f2b2~tplv-k3u1fbpfcp-zoom-1.image)

可以看到，是先给this分配slot，再给方法参数分配slot，再给局部变量分配slot，按顺序来

4.  **然后根据方法体内部定义的变量顺序和作用域来分配slot**
4.  **slot是复用的，以节省栈帧的空间，这种设计可能会影响到系统的垃圾收集行为：**

代码1：

```
public static void main(String[] args) {
    {
	byte[] bs = new byte[2*1024*1024];
    }
    //上面这些是代码块1
	system.gc( );
    //上面这行是代码块2
	System.out.println("totaUMemory===" +Runtime.getRuntime().totalMemory( )/1024.0/1024.0);
    System.out.println("freeMemory==="+Runtime.getRuntime().freeMemory()/1024.0/1024.0);
    System.out.println( "maxMemory==="+Runtime.getRuntime( ).maxMemory( )/1024.0/1024.0);
}
```

假设我们给堆分配8M空间，以上代码输出为：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26abec51632f41cd8209b2e7899d5b07~tplv-k3u1fbpfcp-zoom-1.image)

我们把代码块1和代码块2注释掉之后，输出为：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b6467ba628944d3933e0d54ca9d5aff~tplv-k3u1fbpfcp-zoom-1.image)

只注释代码块2，输出为：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a66a810135004ffaada83d6b596b74bb~tplv-k3u1fbpfcp-zoom-1.image)

说明gc回收并没有把bs的空间回收！

```
public static void main(String[] args) {
    {
	byte[] bs = new byte[2*1024*1024];
    }

    int a =5;
	system.gc( );
    
	System.out.println("totaUMemory===" +Runtime.getRuntime().totalMemory( )/1024.0/1024.0);
    System.out.println("freeMemory==="+Runtime.getRuntime().freeMemory()/1024.0/1024.0);
    System.out.println( "maxMemory==="+Runtime.getRuntime( ).maxMemory( )/1024.0/1024.0);
}
```

上面程序输出为：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17b1c4591ab84d3988ba741fabc50569~tplv-k3u1fbpfcp-zoom-1.image)

说明bs的空间被gc回收了！

**原理** **：** slot0被分配给了args，slot1被分配给了bs，而出了代码块之后bs过了调用空间，此时slot1被分配给了a，所以在垃圾回收的时候，由于bs的空间已经没有任何引用了，所以gc进行了回收，但如果没有a，即使bs出了调用空间，gc也不会回收（为啥啊？），猜测是slot1还是由bs占着，所以没清掉

如果在代码块内定义bs和bs2，则slot1和slot2被分配给这两个，而出了代码块之后，slot1被a占用，所以gc只会回收bs的空间，而不会回收bs2的空间

### 操作数栈

操作数栈∶用来存放方法运行期间，各个指令操作的数据。即一个方法执行期间，每条指令操作的数据会从slot或其他地方加载进栈顶，然后从栈顶取出数据进行运算，将生成的数据放回栈顶

-   操作数栈中元素的数据类型必须和字节码指令的顺序严格匹配
-   虚拟机在实现栈帧的时候可能会做一些优化，让两个栈帧出现部分重叠区域，以存放公用的数据

### 动态连接

每个栈帧持有一个指向运行时常量池中该栈帧所属方法的引用，以支持方法调用过程的动态连接

**什么是动态连接：**

我们知道Class文件的常量池中存 有大量的符号引用，字节码中的方法调用指令就以常量池里指向方法的符号引用作为参数。这些符号 引用一部分会在类加载阶段或者第一次使用的时候就被转化为直接引用，这种转化被称为静态解析。 另外一部分将在每一次运行期间都转化为直接引用，这部分就称为动态连接。 

****

### 方法返回地址

方法执行后返回的地址

### 方法调用

方法调用就是确定具体调用那一个方法，并不涉及方法内部的执行过程， Class文件的编译过程中不包含传统程序语言编译的连接步骤，一切方法调用在Class文件里面存储的都只是符号引用，而不是方法在实际运行时内存布局 中的入口地址（也就是之前说的直接引用）。这个特性给Java带来了更强大的动态扩展能力，但也使得Java方法调用过程变得相对复杂，某些调用需要在类加载期间，甚至到运行期间才能确定目标方法 的直接引用 

1∶部分方法是直接在类加载的解析阶段，就确定了直接引用关系

2∶但是对于实例方法，也称虚方法，因为重载和多态，需要运行期动态委派

### 分派

所谓分派，主要是针对一个方法来讲的，即方法分派。**就是虚拟机如何确定应该执行哪个方法！**

又分成静态分派和动态分派：但不论是静态还是动态，都有动态的特性，这是翻译问题

#### 静态分派︰

所有依赖静态类型来定位方法执行版本的分派方式，比如:重载方法，根据方法参数类型来定位执行方法的版本

```
public class StaticDispatch {
	static abstract class Human {
	}
    
	static class Man extends Human {
	}
    
	static class Woman extends Human {
	}
    
	public void sayHello(Human guy) {
		System.out.println("hello,guy!");
	}
    
	public void sayHello(Man guy) {
		System.out.println("hello,gentleman!");
	}
    
	public void sayHello(Woman guy) {
		System.out.println("hello,lady!");
	}
    
	public static void main(String[] args) {
		Human man = new Man();
		Human woman = new Woman();
		StaticDispatch sr = new StaticDispatch();
		sr.sayHello(man);
		sr.sayHello(woman);
	}
}
```

结果：

```
hello,guy!
hello,guy!
```

为什么呢？让我们先来看一看下面的代码！

Human man = new Man(); 

我们把上面代码中的“Human”称为变量的“**静态类型**”（Static Type），或者叫“外观类 型”（Apparent Type），后面的“Man”则被称为变量的“**实际类型**”（Actual Type）或者叫“运行时类型”（Runtime Type）。静态类型和实际类型在程序中都可能会发生变化，区别是静态类型的变化仅仅 在使用时发生，变量本身的静态类型不会被改变（即临时变化），并且最终的静态类型是在编译期可知的；而实际类型变化的结果在运行期才可确定，编译器在编译程序的时候并不知道一个对象的实际类型是什么。 

```
// 实际类型变化
Human human = (new Random()).nextBoolean() ? new Man() : new Woman();
// 静态类型变化
sr.sayHello((Man) human)
sr.sayHello((Woman) human)
```

对象human的实际类型是可变的，编译期间它完全是个“薛定谔的人”，到底是Man还是Woman，必 须等到程序运行到这行的时候才能确定。而human的静态类型是Human，也可以在使用时（如 sayHello()方法中的强制转型）临时改变这个类型，但这个改变仍是在编译期是可知的，两次sayHello() 方法的调用，在编译期完全可以明确转型的是Man还是Woman。 

回到之前的代码，**虚拟机（或者准确地说是编译器）在重载时是通过参数的静态类型而不是实际类型作为 判定依据的**。由于静态类型在编译期可知，所以在编译阶段，Javac编译器就根据参数的静态类型决定 了会使用哪个重载版本，因此选择了sayHello(Human)作为调用目标，并把这个方法的符号引用写到 main()方法里的两条invokevirtual指令的参数中。 

总结： 所有依赖静态类型来决定方法执行版本的分派动作，都称为静态分派。静态分派的最典型应用表 现就是方法重载。静态分派发生在编译阶段，因此确定静态分派的动作实际上不是由虚拟机来执行 的，这点也是为何一些资料选择把它归入“解析”而不是“分派”的原因 

#### 动态分派：

我们把这种在运行期根据实际类型确定**方法**执行版本的分派过程称为动态分派。比如︰覆盖方法，只有执行之后才知道要调用父类的方法还是子类的方法

invokevirtual指令执行的第一步就是在运行期确定接收者的实际类型，所以两次调用中的 invokevirtual指令并不是把常量池中方法的符号引用解析到直接引用上就结束了，还会根据方法接收者 的实际类型来选择方法版本，这个过程就是Java语言中方法重写的本质。我们把这种在运行期根据实 际类型确定方法执行版本的分派过程称为动态分派。 

既然这种多态性的根源在于虚方法调用指令invokevirtual的执行逻辑，那自然**我们得出的结论就只会对方法有效**，对字段是无效的，因为字段不使用这条指令。事实上，在Java里面只有虚方法存在， 字段永远不可能是虚的，**换句话说，字段永远不参与多态，哪个类的方法访问某个名字的字段时，该名字指的就是这个类能看到的那个字段**

#### 单分派和多分派：

就是按照分派思考的纬度，多于一个的就算多分派，只有一个的称为单分派

比如：父类调用子类覆盖的方法，且该方法是重载的，则有两个维度：1.执行父类还是子类；2.执行哪个重载方法




#### 虚拟机动态分派的实现 

动态分派是执行非常频繁的动作，而且动态分派的方法版本选择过程需要运行时在接收者类型的 方法元数据中搜索合适的目标方法，因此，Java虚拟机实现基于执行性能的考虑，真正运行时一般不 会如此频繁地去反复搜索类型元数据。面对这种情况，一种基础而且常见的优化手段是为类型在方法 区中建立一个虚方法表（Virtual Method Table，也称为vtable，与此对应的，在invokeinterface执行时也 会用到接口方法表——Interface Method Table，简称itable），使用虚方法表索引来代替元数据查找以 提高性能。我们先看看一个虚方法表结构示例，如图8-3所示。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/240509d0339546ba9d61708cc497483b~tplv-k3u1fbpfcp-zoom-1.image)

**虚方法表中存放着各个方法的实际入口地址。如果某个方法在子类中没有被重写，那子类的虚方 法表中的地址入口和父类相同方法的地址入口是一致的，都指向父类的实现入口。如果子类中重写了 这个方法，子类虚方法表中的地址也会被替换为指向子类实现版本的入口地址。** 在图8-3中，Son重写 了来自Father的两个方法（实际上也是Father定义的所有方法），因此Son的方法表没有指向Father类型数据的箭头。但是Son和Father都没有重写来自Object的方法，所以它们的方法表中所有从Object继承来的方法都指向了Object的数据类型。 

## 本地方法栈：

和虚拟机栈所发挥的作用非常相似，区别是： 虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。 **在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。**  与虚拟机栈一样，本地方法栈也会在栈深度溢出或者栈扩展失 败时分别抛出StackOverflowError和OutOfMemoryError异常。 

## 方法区：

方法区是线程共享的， 它用于存储已被虚拟机加载 的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据 ，如果方法区无法满足新的内存分配需求时，将抛出 OutOfMemoryError异常。 

方法区通常和元空间关联在一起，但具体的跟JVM实现和版本有关， 到了 JDK 8，完全废弃了永久代的概念，HotSpot改用与JRockit、J9一样**在本地内存中实现的元空间**（Metaspace）来代替，把JDK 7中永久代还剩余的内容（主要是类型信息）全部移到元空间中。 

注：JDK8之后，方法区中的字符串常量池和静态变量移到了堆中 其他的存放在元空间中

## 运行时常量池：

**是方法区的一部分。**  Class文件中除了有类的版本、字 段、方法、接口等描述信息外，还有一项信息是常量池表（Constant Pool Table），用于存放编译期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。 

运行时常量池相对于Class文件常量池的另外一个重要特征是具备动态性，Java语言并不要求常量 一定只有编译期才能产生，也就是说，并非预置入Class文件中常量池的内容才能进入方法区运行时常 量池，运行期间也可以将新的常量放入池中，这种特性被开发人员利用得比较多的便是String类的 intern()方法。 既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存 时会抛出OutOfMemoryError异常。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0134749d542434881b3a94a78dfe2c8~tplv-k3u1fbpfcp-zoom-1.image)

\


## 直接内存

直接内存（Direct Memory）并**不是虚拟机运行时数据区的一部分**，也不是《Java虚拟机规范》中 定义的内存区域。 

在JDK 1.4中新加入了NIO（New Input/Output）类，引入了一种基于通道（Channel）与缓冲区 （Buffer）的I/O方式，它可以**使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆里面的 DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了 在Java堆和Native堆中来回复制数据。**

-Xmx分配的是虚拟机内存，这不包括直接内存，所以在分配时务必注意，不能让各个内存区域总和大于物理内存限制

直接内存（Direct Memory）的容量大小可通过-XX：MaxDirectMemorySize参数来指定，如果不去指定，则默认与Java堆最大值（由-Xmx指定）一致 

## java堆内存模型

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/927f296806c64a988957f5f1a3b2f182~tplv-k3u1fbpfcp-zoom-1.image)

**注意**：这些区域划分仅仅是一部分垃圾收集器的共同特性或者说设计风格而已，而非某个Java虚拟机具体 实现的固有内存布局，更不是《Java虚拟机规范》里对Java堆的进一步细致划分 。 到了今天，垃圾收集器技术与十年前已不可同日而语，HotSpot里面也出 现了不采用分代设计的新垃圾收集器，再按照上面的提法就有很多需要商榷的地方了。 

新生代用来放新分配的对象，新生代中经过垃圾回收，没有回收掉的对象，被复制到老年代，而对于一些比较大的对象，可能在分配时就放在了老年代

### 对象的创建

当Java虚拟机遇到一条字节码new指令时，首先将去检查这个指令的参数是否能在常量池中定位到 一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程 

在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类加载完成后便可完全确定，分配内存有两种方法：

1.  对于绝对规整（即空闲的内存放在一起，被使用的内存放在一起， 中间放着一个指针作为分界点的指示器 ）的内存，那所分配内存就仅仅是把那个指针向空闲空间方向挪动一段与对象大小相等的距离，这种分配方式称为“指针碰撞”， Serial、ParNew垃圾收集器就是使用这种方法
1.  对于不规整的内存，jvm需要维护一个空闲内存列表，从上面分配内存给对象，CMS收集器就用这种方法，但 为了能在多数情况下分配得更快，CMS设计了一个叫作Linear Allocation Buffer的分配缓冲区，通过空闲列表拿到一大块分配缓冲区之后，在它里面仍然可以使用指针碰撞方式来分配

对象创建不是线程安全的，为了解决这个问题，一般有两种方法：

1.  CAS配上失败 重试的方式保证更新操作的原子性；

<!---->

2.  把内存分配的动作按照线程划分在不同的空间之中进 行，即每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲（Thread Local Allocation Buffer，TLAB），哪个线程要分配内存，就在哪个线程的本地缓冲区中分配，只有本地缓冲区用完 了，分配新的缓存区时才需要同步锁定。虚拟机是否使用TLAB，可以通过-XX：+/-UseTLAB参数来 设定。 

### 对象的内存布局

对象在内存中存储的布局（这里以HotSpot虚拟机为例来说明），分为：**对象头、实例数据和对齐填充**

1.  对象头，包含两个部分︰

( 1 )Mark Word ：存储对象自身的运行数据，如：HashCode、GC分代年龄、锁状态标志等

(2）类型指针：对象指向它的类元数据的指针

2.  实例数据：真正存放对象实例数据的地方
2.  对齐填充：这部分不一定存在，也没有什么特别含义，仅仅是占位符。因为HotSpot要求对象起始地址都是8字节的整数倍，如果不是，就对齐

### 对象的访问定位

在JVM规范中只规定了reference类型是一个指向对象的引用，但没有规定这个引用具体如何去定位、访问堆中对象的具体位置，因此对象的访问方式取决于JVM的实现，目前主流的有：使用句柄或使用指针两种方式

-   **使用句柄**：Java堆中会划分出一块内存来做为句柄池，reference中存储句柄的地址，句柄中存储对象的实例数据和类元数据的地址，如下图所示︰

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2e3f038384f48cf8d76d9f0ea35cac1~tplv-k3u1fbpfcp-zoom-1.image)

使用句柄来访问的最大好处就是reference中存储的是稳定句柄地 址，在对象被移动（垃圾收集时移动对象是非常普遍的行为）时只会改变句柄中的实例数据指针，而 reference本身不需要被修改。 

-   **使用指针**：Java堆中会存放访问类元数据的地址，reference存储的就直接是对象的地址，如下图所示︰

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08be6603028d4707b97b841c24c428af~tplv-k3u1fbpfcp-zoom-1.image)

使用直接指针来访问最大的好处就是速度更快，它节省了一次指针定位的时间开销，由于对象访 问在Java中非常频繁，因此这类开销积少成多也是一项极为可观的执行成本。HotSpot采用的就是这种方法

# 各种参数配置

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8fc430f99a24e3b987dd697a12b32cb~tplv-k3u1fbpfcp-zoom-1.image)

在这里可以设置运行时的一些参数

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/506ec5af6e324602b37b078f61fccc2c~tplv-k3u1fbpfcp-zoom-1.image)

## Trace追踪参数

-XX:+PrintGC 输出GC日志

-XX:+PrintGCDetails 输出GC的详细日志

-XX:+PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式）

-XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800）

-XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息

-Xloggc:../logs/gc.log 日志文件的输出路径

## 堆参数（重点）

**-Xmx：** JVM最大可用内存，默认为物理内存的1/4

**-Xmx100m**：设置JVM最大可用内存为100M，

**-Xms：** JVM初始内存，默认为物理内存的1/64\
**-Xms100m**：设置JVM促使内存为100m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。

```
   @Test
    public void testAopAnno() {
        System.out.println(Runtime.getRuntime().totalMemory()/1024/1024);//输出jvm初始内存
        System.out.println(Runtime.getRuntime().freeMemory()/1024/1024);//输出jvm当前可用内存
        System.out.println(Runtime.getRuntime().maxMemory()/1024/1024);//输出jvm最大可用内存
    }
}
```

设置**-Xmx100m -Xms80m**后：77 69 89

与设定不同的原因大概是**gc收集器算法会给一个survivor区分配内存，而maxMemory()返回的内存大小不包括survivor区的内存**，使用g1算法收集器的话，就一样了

**-XX:+UseG1GC ：** 80 76 100

**-Xmn：** 新生代大小，默认为整个堆的3/8

test: -ea -Xmx100m -Xms80m -XX:+UseConcMarkSweepGC -Xmn10m -XX:+PrintGCDetails

```
Heap
 par new generation   total 9216K, used 7232K [0x00000000f9c00000, 0x00000000fa600000, 0x00000000fa600000)
  eden space 8192K,  88% used [0x00000000f9c00000, 0x00000000fa310038, 0x00000000fa400000)
  from space 1024K,   0% used [0x00000000fa400000, 0x00000000fa400000, 0x00000000fa500000)
  to   space 1024K,   0% used [0x00000000fa500000, 0x00000000fa500000, 0x00000000fa600000)
 concurrent mark-sweep generation total 71680K, used 0K [0x00000000fa600000, 0x00000000fec00000, 0x0000000100000000)
 Metaspace       used 5060K, capacity 5264K, committed 5504K, reserved 1056768K
  class space    used 587K, capacity 627K, committed 640K, reserved 1048576K

Process finished with exit code 0
```

par new generation 就是新生代

concurrent mark-sweep 是老生代

**-XX:NewRatio=ratioSets** the ratio between young and old generation sies. By defult, ths option is set to 2. The following example shows how to set the young-to-old ratio to 1:

-XX : NewRatio=1

**-Xx:SurvivorRatio** : Eden区和Survivor区的大小比值，设置为8，则两个Survivor区与一个Eden区的比值为2:8，一个Survivor占整个新生的1/10

-XX:+HeapDumpOnOutOfMemoryError : OOM时导出堆到文件

-XX:+HeapDumpPath :导出OOM的路径

-XX:OnOutOfMemoryError :在OOM时，执行一个脚本

## 栈参数

-Xss：通常只有几百K，决定了函数调用的深度

## 元空间参数

-XX:MetaspaceSize:初始空间大小

-XX:MaxMetaspaceSize:最大空间，默认是没有限制的

-XX:MinMetaspaceFreeRatio :在GC之后，最小的Metaspace剩余空间容量的百分比

-XX:MaxMetaspaceFreeRatio :在GC之后，最大的Metaspace剩余空间容量的百分比

# 工作内存和主内存

Java内存模型的主要目标是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样底层细节。此处的变量与Java编程时所说的变量不一样，指包括了实例字段、静态字段和构成数组对象的元素，但是不包括局部变量与方法参数，后者是线程私有的，不会被共享。

Java内存模型中规定了所有的变量都存储在主内存中（可以与物理硬件的内存类比），每条线程还有自己的工作内存（可以与前面将的处理器的高速缓存类比），线程的工作内存中保存了该线程使用到的变量到主内存副本拷贝（注：并不是整个对象的拷贝，而是引用或部分字段的拷贝），线程对变量的所有操作（读取、赋值）都必须在工作内存中进行，而不能直接读写主内存中的变量。不同线程之间无法直接访问对方工作内存中的变量，线程间变量值的传递均需要在主内存来完成，线程、主内存和工作内存的交互关系如下图所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6b34c9baaac4535b592d3affe80e73c~tplv-k3u1fbpfcp-zoom-1.image)

这里的主内存、工作内存与Java内存区域的Java堆、栈、方法区不是同一层次内存划分。

关于主内存与工作内存之间的具体交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存同步到主内存之间的实现细节，Java内存模型定义了以下八种操作来完成，且这些必须是原子的、不可再分的：

lock（锁定）：作用于主内存的变量，把一个变量标识为一条线程独占状态。

unlock（解锁）：作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。

read（读取）：作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用

load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。

use（使用）：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。

assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。

store（存储）：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。

write（写入）：作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中。

如果要把一个变量从主内存中复制到工作内存，就需要按顺寻地执行read和load操作，如果把变量从工作内存中同步回主内存中，就要按顺序地执行store和write操作。Java内存模型只要求上述操作必须按顺序执行，而没有保证必须是连续执行。也就是read和load之间，store和write之间是可以插入其他指令的，如对主内存中的变量a、b进行访问时，可能的顺序是read a，read b，load b， load a。Java内存模型还规定了在执行上述八种基本操作时，必须满足如下规则：

-   **不允许read和load、store和write操作之一单独出现**
-   **不允许一个线程丢弃它的最近assign的操作，即变量在工作内存中改变了之后必须同步到主内存中。**

<!---->

-   **不允许一个线程无原因地（没有发生过任何assign操作）把数据从工作内存同步回主内存中。**
-   **一个新的变量只能在主内存中诞生，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量。即就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。**

<!---->

-   **一个变量在同一时刻只允许一条线程对其进行lock操作，lock和unlock必须成对出现**
-   **如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行load或assign操作初始化变量的值**

<!---->

-   **如果一个变量事先没有被lock操作锁定，则不允许对它执行unlock操作；也不允许去unlock一个被其他线程锁定的变量。**
-   **对一个变量执行unlock操作之前，必须先把此变量同步到主内存中（执行store和write操作）。**

