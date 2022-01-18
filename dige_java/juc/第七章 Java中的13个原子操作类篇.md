## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂
# 基本介绍

Java从JDK 1.5开始提供了java.util.concurrent.atomic包(以下简称Atomic包)，这个包中的原子操作类提供了一种用法简单、性能高效、线程安全地更新一个变量的方式。

因为变量的类型有很多种，所以在Atomic包里一共提供了13个类，属于4种类型的原子更新方式，分别是原子更新基本类型、原子更新数组、原子更新引用和原子更新属性（字段)。Atomic包里的类基本都是使用Unsafe实现的包装类。

# 原子更新基本类型类

使用原子的方式更新基本类型，Atomic包提供了以下3个类。

-   AtomicBoolean：原子更新布尔类型。
-   AtomicInteger：原子更新整型。

<!---->

-   AtomicLong：原子更新长整型。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24f7631d1cea41ee97d8417638fbf12d~tplv-k3u1fbpfcp-zoom-1.image)

例：

```
import java.util.concurrent.atomic.AtomicInteger;
public class AtomicIntegerTest {
 static AtomicInteger ai = new AtomicInteger(1);
 public static void main(String[] args) {
 System.out.println(ai.getAndIncrement());
 System.out.println(ai.get());
 }
}
//输出：1 2
```

## getAndIncrement 源码分析

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a2765df273b42158f95ab78652cef97~tplv-k3u1fbpfcp-zoom-1.image)

源码中for循环体的第一步先取得AtomicInteger里存储的数值，第二步对AtomicInteger的当前数值进行加1操作，关键的第三步调用compareAndSet方法来进行原子更新操作，该方法先检查当前数值是否等于current，等于意味着AtomicInteger的值没有被其他线程修改过，则将Atomiclnteger的当前数值更新成next的值，如果不等comparcAndSet方法会返回false，程序会进入for循环重新进行compareAndSet操作。
## Unsafe源码

Atomic包提供了3种基本类型的原子更新，但是Java的基本类型里还有char、float和double等。那么问题来了，如何原子的更新其他的基本类型呢?Atomic包里的类基本都是使用Unsafe实现的，让我们一起看一下Unsafe的源码，如代码清单7-3所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7b4044e05904ca685f1e411566e0af5~tplv-k3u1fbpfcp-zoom-1.image)

通过代码，我们发现Unsafe只提供了3种CAS方法: compareAndSwapObject、compareAndSwapInt和

compareAndSwapLong，再看AtomicBoolean源码，发现它是**先把Boolean转换成整型，再使用compareAndSwapInt进行CAS，所以原子更新char、float和double变量也可以用类似的思路来实现。**

# 原子更新数组

通过原子的方式更新数组里的某个元素，Atomic包提供了以下4个类。

-   AtomicIntegerArray:原子更新整型数组里的元素。
-   AtomicLongArray:原子更新长整型数组里的元素。

<!---->

-   AtomicReferenceArray:原子更新引用类型数组里的元素。
-   AtomicIntegerArray类主要是提供原子的方式更新数组里的整型，其常用方法如下：

<!---->

-   -   int addAndGet (int i，int delta) :以原子方式将输入值与数组中索引i的元素相加。
    -   boolean compareAndSet (int i，int expect，int update):如果当前值等于预期值，则以原子方式将数组位置i的元素设置成update值。

例：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1180434e0454e239b3f8c2530725c89~tplv-k3u1fbpfcp-zoom-1.image)

输出：3 1

需要注意的是，数组value通过构造方法传递进去，然后AtomicIntegerArray会将当前数组复制一份，所以当AtomicIntegerArray对内部的数组元素进行修改时，不会影响传入的数组。

# 原子更新引用类型

原子更新基本类型的AtomicInteger，只能更新一个变量，如果要原子更新多个变量，就需要使用这个原子更新引用类型提供的类。Atomic包提供了以下3个类。

-   AtomicReference: 原子更新引用类型。
-   AtomicReferenceFieldUpdater : 原子更新引用类型里的字段。

<!---->

-   AtomicMarkableReference: 原子更新带有标记位的引用类型。可以原子更新一个布尔类型的标记位和引用类型。构造方法是AtomicMarkableReference (V initialRef，boolean initialMark) 。

```
public class AtomicReferenceTest {
 public static AtomicReference<user> atomicUserRef = new AtomicReference<user>();
 public static void main(String[] args) {
 	User user = new User("conan"҅ 15);
	 atomicUserRef.set(user);
	 User updateUser = new User("Shinichi"҅ 17);
	 atomicUserRef.compareAndSet(user҅ updateUser);
	 System.out.println(atomicUserRef.get().getName());
     System.out.println(atomicUserRef.get().getOld());
 }
 static class User {
	 private String name;
	 private int old;
 	public User(String name҅ int old) {
 	this.name = name;
 	this.old = old;
 }
 public String getName() {
	 return name;
 }
 public int getOld() {
 	return old;
 }
 }
}
//输出 Shinichi 17
```

# 原子更新字段类

如果需原子地更新某个类里的某个字段时，就需要使用原子更新字段类，Atomic包提供了以下3个类进行原子字段更新。

-   AtomicIntegerFieldUpdater: 原子更新整型的字段的更新器。
-   AtomicLongFieldUpdater : 原子更新长整型字段的更新器。

<!---->

-   AtomicStampedReference: 原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于原子的更新数据和数据的版本号，可以解决使用CAS进行原子更新时可能出现的ABA问题。

```
public class AtomicIntegerFieldUpdaterTest {
	//创建原子更新器，并设置需要更新的对象类和对象的属性
 private static AtomicIntegerFieldUpdater<User> a = AtomicIntegerFieldUpdater.
 newUpdater(User.class҅ "old");
 public static void main(String[] args) {

 User conan = new User("conan"҅ 10);

 System.out.println(a.getAndIncrement(conan));

 System.out.println(a.get(conan));
 }
 public static class User {
 private String name;
 public volatile int old;
 public User(String name҅ int old) {
 this.name = name;
 this.old = old;
 }
 public String getName() {
 return name;
 }
 public int getOld() {
 return old;
 }
 }
}
//输出 10 11
```

若有收获，就点个赞吧

