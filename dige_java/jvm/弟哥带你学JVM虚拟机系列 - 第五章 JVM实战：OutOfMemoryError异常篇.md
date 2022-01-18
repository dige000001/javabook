## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 堆溢出

-XX：+HeapDumpOnOutOf-MemoryError可以让虚拟机 在出现内存溢出异常的时候Dump出当前的内存堆转储快照以便进行事后分析 

```
/**
* VM Args：-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
* @author zzm
*/
public class HeapOOM {
	static class OOMObject {
	}
	public static void main(String[] args) {
		List<OOMObject> list = new ArrayList<OOMObject>();
		while (true) {
			list.add(new OOMObject());
		}
	}
}
```

```
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid3404.hprof ...
Heap dump file created [22045981 bytes in 0.663 secs]
```

Java堆内存的OutOfMemoryError异常是实际应用中最常见的内存溢出异常情况。出现Java堆内存 溢出时，异常堆栈信息“java.lang.OutOfMemoryError”会跟随进一步提示“Java heap space”。 

要解决这个内存区域的异常，常规的处理方法是首先通过内存映像分析工具（如Eclipse Memory Analyzer）对Dump出来的堆转储快照进行分析。第一步首先应确认内存中导致OOM的对象是否是必 要的，也就是要先分清楚到底是出现了内存泄漏（Memory Leak）还是内存溢出（Memory Overflow）。 

P.S:**内存泄漏：** 指分配了一块空间后一直不回收，但是该空间已经不会再被使用了，导致可用空间不足

如果是内存泄漏，可进一步通过工具查看泄漏对象到GC Roots的引用链，找到泄漏对象是通过怎 样的引用路径、与哪些GC Roots相关联，才导致垃圾收集器无法回收它们，根据泄漏对象的类型信息 以及它到GC Roots引用链的信息，一般可以比较准确地定位到这些对象创建的位置，进而找出产生内 存泄漏的代码的具体位置 

如果不是内存泄漏，换句话说就是内存中的对象确实都是必须存活的，那就应当检查Java虚拟机 的堆参数（-Xmx与-Xms）设置，与机器的内存对比，看看是否还有向上调整的空间。再从代码上检查 是否存在某些对象生命周期过长、持有状态时间过长、存储结构设计不合理等情况，尽量减少程序运 行期的内存消耗。 

# 虚拟机栈和本地方法栈溢出 

由于HotSpot虚拟机中并不区分虚拟机栈和本地方法栈，因此对于HotSpot来说，-Xoss参数（设置 本地方法栈大小）虽然存在，但实际上是没有任何效果的，栈容量只能由-Xss参数来设定。关于虚拟 机栈和本地方法栈，在《Java虚拟机规范》中描述了两种异常： 1）如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常。 2）如果虚拟机的栈内存允许动态扩展，当扩展栈容量无法申请到足够的内存时，将抛出 OutOfMemoryError异常。 

```
/**
* VM Args：-Xss128k
* @author zzm
*/
public class JavaVMStackSOF {
	private int stackLength = 1;
	public void stackLeak() {
		stackLength++;
		stackLeak();
	}
	public static void main(String[] args) throws Throwable {
		JavaVMStackSOF oom = new JavaVMStackSOF();
		try {
			oom.stackLeak();
		} catch (Throwable e) {
			System.out.println("stack length:" + oom.stackLength);
			throw e;
		}
	}
}
```

```
stack length:2402
Exception in thread "main" java.lang.StackOverflowError
at org.fenixsoft.oom. JavaVMStackSOF.leak(JavaVMStackSOF.java:20)
at org.fenixsoft.oom. JavaVMStackSOF.leak(JavaVMStackSOF.java:21)
at org.fenixsoft.oom. JavaVMStackSOF.leak(JavaVMStackSOF.java:21)
```

# 方法区和运行时常量池溢出 

由于运行时常量池是方法区的一部分，所以这两个区域的溢出测试可以放到一起进行。 

## 运行时常量池溢出测试：JDK6及以前

HotSpot从JDK 7开始逐步“去永久代”的计划，并在JDK 8中完全使用元空间来代替永久代 ， 原本存放在永久代的字符串常量池被移至Java堆之中 ，所以这个测试我们要用JDK6来进行

String::intern()是一个本地方法，它的作用是如果字符串常量池中已经包含一个等于此String对象的 字符串，则返回代表池中这个字符串的String对象的引用；否则，会将此String对象包含的字符串添加 到常量池中，并且返回此String对象的引用。在JDK 6或更早之前的HotSpot虚拟机中，常量池都是分配 在永久代中，我们可以通过 -XX：PermSize和-XX：MaxPermSize限制永久代的大小，即可间接限制其 中常量池的容量 

```
/**
* VM Args：-XX:PermSize=6M -XX:MaxPermSize=6M
* Using JDK 6
* @author zzm
*/
public class RuntimeConstantPoolOOM {
	public static void main(String[] args) {
		// 使用Set保持着常量池引用，避免Full GC回收常量池行为
		Set<String> set = new HashSet<String>();
		// 在short范围内足以让6MB的PermSize产生OOM了
		short i = 0;
		while (true) {
			set.add(String.valueOf(i++).intern());
		}
	}
}
```

```
Exception in thread "main" java.lang.OutOfMemoryError: PermGen space
at java.lang.String.intern(Native Method)
at org.fenixsoft.oom.RuntimeConstantPoolOOM.main(RuntimeConstantPoolOOM.java: 18)
```

\


## 方法去溢出测试：JDK7

我们再来看看方法区的其他部分的内容，方法区的主要职责是用于存放类型的相关信息，如类 名、访问修饰符、常量池、字段描述、方法描述等。对于这部分区域的测试，基本的思路是运行时产 生大量的类去填满方法区 这里 借助了CGLib直接操作字节码运行时生成了大量的动态类 

值得特别注意的是，我们在这个例子中模拟的场景并非纯粹是一个实验，类似这样的代码确实可 能会出现在实际应用中：当前的很多主流框架，如Spring、Hibernate对类进行增强时，都会使用到 CGLib这类字节码技术，当增强的类越多，就需要越大的方法区以保证动态生成的新类型可以载入内 存。另外，很多运行于Java虚拟机上的动态语言（例如Groovy等）通常都会持续创建新类型来支撑语 言的动态性，随着这类动态语言的流行，与测试代码相似的溢出场景也越来越容易遇到。 

```
/**
* VM Args：-XX:PermSize=10M -XX:MaxPermSize=10M
* @author zzm
*/
public class JavaMethodAreaOOM {
    
    static class OOMObject {
	}
    
	public static void main(String[] args) {
		while (true) {
			Enhancer enhancer = new Enhancer();
			enhancer.setSuperclass(OOMObject.class);
			enhancer.setUseCache(false);
			enhancer.setCallback(new MethodInterceptor() {
				public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
					return proxy.invokeSuper(obj, args);
				}
			});
			enhancer.create();
		}
	}
}
```

```
Caused by: java.lang.OutOfMemoryError: PermGen space
at java.lang.ClassLoader.defineClass1(Native Method)
at java.lang.ClassLoader.defineClassCond(ClassLoader.java:632)
at java.lang.ClassLoader.defineClass(ClassLoader.java:616)
... 8 more
```

方法区溢出也是一种常见的内存溢出异常，一个类如果要被垃圾收集器回收，要达成的条件是比 较苛刻的。在经常运行时生成大量动态类的应用场景里，就应该特别关注这些类的回收状况。这类场 景除了之前提到的程序使用了CGLib字节码增强和动态语言外，常见的还有：大量JSP或动态产生JSP 文件的应用（JSP第一次运行时需要编译为Java类）、基于OSGi的应用（即使是同一个类文件，被不同 的加载器加载也会视为不同的类）等。 

## JDK8的防御措施

在JDK 8以后，永久代便完全退出了历史舞台，元空间作为其替代者登场。在默认设置下，前面 列举的那些正常的动态创建新类型的测试用例已经很难再迫使虚拟机产生方法区的溢出异常了。不过 为了让使用者有预防实际应用里出现类似于上述代码那样的破坏性的操作，HotSpot还是提供了一 些参数作为元空间的防御措施，主要包括： 

-   **-XX：MaxMetaspaceSize：** 设置元空间最大值，默认是-1，即不限制，或者说只受限于本地内存 大小。 
-   -**XX：MetaspaceSize：** 指定元空间的初始空间大小，以字节为单位，达到该值就会触发垃圾收集 进行类型卸载，同时收集器会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放 了很少的空间，那么在不超过-XX：MaxMetaspaceSize（如果设置了的话）的情况下，适当提高该 值。 

<!---->

-   **-XX：MinMetaspaceFreeRatio：** 作用是在垃圾收集之后控制最小的元空间剩余容量的百分比，可 减少因为元空间不足导致的垃圾收集的频率。类似的还有-XX：Max-MetaspaceFreeRatio，用于控制最 大的元空间剩余容量的百分比。 

\


# 本机直接内存溢出 

直接内存（Direct Memory）的容量大小可通过-XX：MaxDirectMemorySize参数来指定，如果不 去指定，则默认与Java堆最大值（由-Xmx指定）一致 

本测试中， 通过反射获取Unsafe实例进行内存分配（Unsafe类的getUnsafe()方法指定只有引导类加载器才会返回实 例，体现了设计者希望只有虚拟机标准类库里面的类才能使用Unsafe的功能，在JDK 10时才将Unsafe 的部分功能通过VarHandle开放给外部使用），因为虽然使用DirectByteBuffer分配内存也会抛出内存溢 出异常，但它抛出异常时并没有真正向操作系统申请分配内存，而是通过计算得知内存无法分配就会 在代码里手动抛出溢出异常，真正申请分配内存的方法是Unsafe::allocateMemory()。 

```
/**
* VM Args：-Xmx20M -XX:MaxDirectMemorySize=10M
* @author zzm
*/
public class DirectMemoryOOM {
	private static final int _1MB = 1024 * 1024;
	public static void main(String[] args) throws Exception {
		Field unsafeField = Unsafe.class.getDeclaredFields()[0];
		unsafeField.setAccessible(true);
		Unsafe unsafe = (Unsafe) unsafeField.get(null);
		while (true) {
			unsafe.allocateMemory(_1MB);
		}
	}
}
```

```
Exception in thread "main" java.lang.OutOfMemoryError
at sun.misc.Unsafe.allocateMemory(Native Method)
at org.fenixsoft.oom.DMOOM.main(DMOOM.java:20)
```

由直接内存导致的内存溢出，一个明显的特征是在Heap Dump文件中不会看见有什么明显的异常 情况，如果读者发现内存溢出之后产生的Dump文件很小，而程序中又直接或间接使用了 DirectMemory（典型的间接使用就是NIO），那就可以考虑重点检查一下直接内存方面的原因了

