## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 类的生命周期：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62c69250f6ed40bdb05eddca93eb2381~tplv-k3u1fbpfcp-zoom-1.image)



**加载**:查找并加载类文件的二进制数据

**连接:** 就是将已经读入内存的类的二进制数据合并到JVM运行时环境中去，包含如下几个步骤︰

1)验证∶确保被加载类的正确性

2准备∶为类的静态变量分配内存，并初始化它们（这里的初始化是赋默认值，比如static int a =5; 在这个阶段赋的

值是0）

3)解析∶把常量池中的符号引用转换成直接引用

**初始化∶**为类的静态变量赋初始值，如给a赋5

注：这里的顺序指的是“开始”的顺序，它们不是前一个结束后一个才能开始的关系，有可能在前一个流程中就会开始下一个流程。

## 类的加载

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a13333a7784e4dc384fe177aa8f73618~tplv-k3u1fbpfcp-zoom-1.image)

《Java虚拟机规范》对这三点要求其实并不是特别具体，留给虚拟机实现与Java应用的灵活度都是 相当大的。例如“通过一个类的全限定名来获取定义此类的二进制字节流”这条规则，它并没有指明二 进制字节流必须得从某个Class文件中获取，确切地说是根本没有指明要从哪里获取、如何获取。 

相对于类加载过程的其他阶段，非数组类型的加载阶段（准确地说，是加载阶段中获取类的二进制字节流的动作）是开发人员可控性最强的阶段。加载阶段既可以使用Java虚拟机里内置的引导类加载器来完成，也可以由用户自定义的类加载器去完成，开发人员通过定义自己的类加载器去控制字节 流的获取方式（重写一个类加载器的findClass()或loadClass()方法），实现根据自己的想法来赋予应用 程序获取运行代码的动态性。 

对于数组类而言，情况就有所不同，数组类本身不通过类加载器创建，它是由Java虚拟机直接在内存中动态构造出来的。但数组类与类加载器仍然有很密切的关系，因为数组类的元素类型（Element Type，指的是数组去掉所有维度的类型）最终还是要靠类加载器来完成加载，一个数组类（下面简称为C）创建过程遵循以下规则： 

-   如果数组的**组件类型（Component Type，指的是数组去掉一个维度的类型，注意和前面的元素类型区分开来）** 是引用类型，那就递归采用本节中定义的加载过程去加载这个组件类型，数组C将被标识在加载该组件类型的类加载器的类名称空间上（这点很重要，一个类型必须与类加载器一起确定唯一性）。
-   如果数组的组件类型不是引用类型（例如int[]数组的组件类型为int），Java虚拟机将会把数组C 标记为与引导类加载器关联。 

<!---->

-   数组类的可访问性与它的组件类型的可访问性一致，如果组件类型不是引用类型，它的数组类的可访问性将默认为public，可被所有的类和接口访问到。 



## 类连接

### 验证

验证是连接阶段的第一步，这一阶段的目的是确保Class文件的字节流中包含的信息符合《Java虚拟机规范》的全部约束要求，保证这些信息被当作代码运行后不会危害虚拟机自身的安全。 验证阶段大致上会完成下面四个阶段的检验动作： 

1.  **类文件结构检查:**  该验证阶段的主要目的是保证输入的字节流能正确地解析并存储于方法区之内，格式上符 合描述一个Java类型信息的要求。这阶段的验证是基于二进制字节流进行的，只有通过了这个阶段的 验证之后，这段字节流才被允许进入Java虚拟机内存的方法区中进行存储，所以后面的三个验证阶段全部是基于方法区的存储结构上进行的，不会再直接读取、操作字节流了。 

```
·是否以魔数0xCAFEBABE开头。
·主、次版本号是否在当前Java虚拟机接受范围之内。
·常量池的常量中是否有不被支持的常量类型（检查常量tag标志）。
·指向常量的各种索引值中是否有指向不存在的常量或不符合类型的常量。
·CONSTANT_Utf8_info型的常量中是否有不符合UTF-8编码的数据。
·Class文件中各个部分及文件本身是否有被删除的或附加的其他信息。
·…
```



2.  **元数据验证:**  第二阶段的主要目的是对类的元数据信息进行语义校验，保证不存在与《Java语言规范》定义相 悖的元数据信息。 

```
·这个类是否有父类（除了java.lang.Object之外，所有的类都应当有父类）。
·这个类的父类是否继承了不允许被继承的类（被final修饰的类）。
·如果这个类不是抽象类，是否实现了其父类或接口之中要求实现的所有方法。
·类中的字段、方法是否与父类产生矛盾（例如覆盖了父类的final字段，或者出现不符合规则的方
法重载，例如方法参数都一致，但返回值类型却不同等）。
·……
```



3.  **字节码验证:** 通过对数据流和控制流进行分析，确保程序语义是合法和符合逻辑的。这里**主要对方法体进行校验**

```
·保证任意时刻操作数栈的数据类型与指令代码序列都能配合工作，例如不会出现类似于“在操作
  栈放置了一个int类型的数据，使用时却按long类型来加载入本地变量表中”这样的情况。
·保证任何跳转指令都不会跳转到方法体以外的字节码指令上。
·保证方法体中的类型转换总是有效的，例如可以把一个子类对象赋值给父类数据类型，这是安全
的，但是把父类对象赋值给子类数据类型，甚至把对象赋值给与它毫无继承关系、完全不相干的一个
数据类型，则是危险和不合法的。
·……
```

由于数据流分析和控制流分析的高度复杂性，Java虚拟机的设计团队为了避免过多的执行时间消耗在字节码验证阶段中，在JDK 6之后的Javac编译器和Java虚拟机里进行了一项联合优化，把尽可能 多的校验辅助措施挪到Javac编译器里进行。具体做法是给方法体Code属性的属性表中新增加了一项名 为“StackMapTable”的新属性，这项属性描述了方法体所有的基本块（Basic Block，指按照控制流拆分 的代码块）开始时本地变量表和操作栈应有的状态，在字节码验证期间，Java虚拟机就不需要根据程序推导这些状态的合法性，只需要检查StackMapTable属性中的记录是否合法即可。这样就将字节码验证的类型推导转变为类型检查，从而节省了大量校验时间。理论上StackMapTable属性也存在错误或被篡改的可能，所以是否有可能在恶意篡改了Code属性的同时，也生成相应的StackMapTable属性来骗过虚拟机的类型校验，则是虚拟机设计者们需要仔细思考的问题。 



4.  **符号引用验证**∶对类自身以外的信息，也就是常量池中的各种符号引用（不包括字面量，如final常量。在Java代码中，就用“字面量”来标识数据。简单地说，字面量就是数据，而不是变量），进行匹配校验，这一步在“解析”流程中完成。符号引用验证可以看作是对类自身以外（常量池中的各种符号引用）的各类信息进行匹配性校验，通俗来说就是，该类是否缺少或者被禁止访问它依赖的某些外部类、方法、字段等资源。本阶段通常需要校验下列内容： 

```
·符号引用中通过字符串描述的全限定名是否能找到对应的类。
·在指定类中是否存在符合方法的字段描述符及简单名称所描述的方法和字段。
·符号引用中的类、字段、方法的可访问性（private、protected、public、<package>）是否可被当
前类访问。
·……
```



### 准备

为类的**静态变量**分配内存，并初始化它们（**这里的初始化是赋默认值，比如static int a =5; 在这个阶段赋的值是0而不是5**），这些静态变量在jdk8之后和java类一起存放在堆中

如果类字段的字段属性表中存在ConstantValue属性，那在准备阶段变量值就会被初始化为ConstantValue属性所指定的初始值， 如 public static final int value = 123; 编译时Javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会根据Con-stantValue的设置 将value赋值为123。 

还有一个容易产生混淆的概念需要着重强调，这时候**进行内存分配的仅包括类变量，而不包括实例变量**，实例变量将会在对象实例化时随着对象一起分配在Java堆中。

-   **类变量也叫静态变量**，也就是在变量前加了static 的变量；
-   实例变量也叫对象变量，即没加static 的变量；

### 解析

所谓解析就是把**常量池**中的符号引用转换成直接引用的过程，包括:

**符号引用:** 以一组无歧义的符号来描述所引用的目标，与虚拟机的实现无关。

**直接引用:** 直接指向目标的指针、相对偏移量、或是能间接定位到目标的句柄，是和虚拟机实现相关的。

主要针对∶类、接口、字段、类方法、接口方法、方法类型、方法句柄、调用点限定符

**具体的解析过程看《深入理解JVM》P372，这里说几个简单的：**

****

#### 1.类或接口的解析

假设当前代码所处的类为D，如果要把一个从未解析过的符号引用N解析为一个类或接口C的直接引用，那虚拟机完成整个解析的过程需要包括以下3个步骤：

1）如果C不是一个数组类型，那虚拟机将会把代表N的全限定名传递给D的类加载器去加载这个 类C。在加载过程中，由于元数据验证、字节码验证的需要，又可能触发其他相关类的加载动作，例如加载这个类的父类或实现的接口。一旦这个加载过程出现了任何异常，解析过程就将宣告失败。

2）如果C是一个数组类型，并且数组的元素类型为对象，也就是N的描述符会是类 似“[Ljava/lang/Integer”的形式，那将会按照第一点的规则加载数组元素类型。如果N的描述符如前面所假设的形式，需要加载的元素类型就是“java.lang.Integer”，接着由虚拟机生成一个代表该数组维度和元素的数组对象。 

3）如果上面两步没有出现任何异常，那么C在虚拟机中实际上已经成为一个有效的类或接口了， 但在解析完成前还要进行符号引用验证，确认D是否具备对C的访问权限。如果发现不具备访问权限， 将抛出java.lang.IllegalAccessError异常。 

#### 2.字段解析 

要解析一个未被解析过的字段符号引用，首先将会对字段表内class_index项中索引的 CONSTANT_Class_info符号引用进行解析，也就是字段所属的类或接口的符号引用。如果在解析这个类或接口符号引用的过程中出现了任何异常，都会导致字段符号引用解析的失败。如果解析成功完成，那把这个字段所属的类或接口用C表示，《Java虚拟机规范》要求按照如下步骤对C进行后续字段的搜索：

1）如果C本身就包含了简单名称和字段描述符都与目标相匹配的字段，则返回这个字段的直接引 用，查找结束。 

2）否则，如果在C中实现了接口，将会按照继承关系从下往上递归搜索各个接口和它的父接口， 如果接口中包含了简单名称和字段描述符都与目标相匹配的字段，则返回这个字段的直接引用，查找 结束。

3）否则，如果C不是java.lang.Object的话，将会按照继承关系从下往上递归搜索其父类，如果在父 类中包含了简单名称和字段描述符都与目标相匹配的字段，则返回这个字段的直接引用，查找结束。

4）否则，查找失败，抛出java.lang.NoSuchFieldError异常。 如果查找过程成功返回了引用，将会对这个字段进行权限验证，如果发现不具备对字段的访问权 限，将抛出java.lang.IllegalAccessError异常。 

以上解析规则能够确保Java虚拟机获得字段唯一的解析结果 

#### 3.方法解析 

方法解析的第一个步骤与字段解析一样，也是需要先解析出方法表的class_index项中索引的方法所属的类或接口的符号引用，如果解析成功，那么我们依然用C表示这个类，接下来虚拟机将会按 照如下步骤进行后续的方法搜索： 

1）由于Class文件格式中类的方法和接口的方法符号引用的常量类型定义是分开的，如果在类的 方法表中发现class_index中索引的C是个接口的话，那就直接抛出java.lang.IncompatibleClassChangeError 异常。 

2）如果通过了第一步，在类C中查找是否有简单名称和描述符都与目标相匹配的方法，如果有则 返回这个方法的直接引用，查找结束。 

3）否则，在类C的父类中递归查找是否有简单名称和描述符都与目标相匹配的方法，如果有则返 回这个方法的直接引用，查找结束。

4）否则，在类C实现的接口列表及它们的父接口之中递归查找是否有简单名称和描述符都与目标 相匹配的方法，如果存在匹配的方法，说明类C是一个抽象类，这时候查找结束，抛出 java.lang.AbstractMethodError异常。 

5）否则，宣告方法查找失败，抛出java.lang.NoSuchMethodError。 

最后，如果查找过程成功返回了直接引用，将会对这个方法进行权限验证，如果发现不具备对此 方法的访问权限，将抛出java.lang.IllegalAccessError异常。 

#### 4.接口方法解析 

接口方法也是需要先解析出接口方法表的class_index项中索引的方法所属的类或接口的符号引 用，如果解析成功，依然用C表示这个接口，接下来虚拟机将会按照如下步骤进行后续的接口方法搜 索：

1）与类的方法解析相反，如果在接口方法表中发现class_index中的索引C是个类而不是接口，那 么就直接抛出java.lang.IncompatibleClassChangeError异常。

2）否则，在接口C中查找是否有简单名称和描述符都与目标相匹配的方法，如果有则返回这个方 法的直接引用，查找结束。

3）否则，在接口C的父接口中递归查找，直到java.lang.Object类（接口方法的查找范围也会包括 Object类中的方法）为止，看是否有简单名称和描述符都与目标相匹配的方法，如果有则返回这个方 法的直接引用，查找结束。

4）对于规则3，由于Java的接口允许多重继承，如果C的不同父接口中存有多个简单名称和描述符 都与目标相匹配的方法，那将会从这多个方法中返回其中一个并结束查找，《Java虚拟机规范》中并 没有进一步规则约束应该返回哪一个接口方法。但与之前字段查找类似地，不同发行商实现的Javac编 译器有可能会按照更严格的约束拒绝编译这种代码来避免不确定性。

5）否则，宣告方法查找失败，抛出java.lang.NoSuchMethodError异常。 



## 类的初始化

**类的初始化就是为类的静态变量赋初始值，或者说是执行类构造器<clinit>方法的过程：**

1.  如果类还没有加载和连接，就先加载和连接
1.  如果类存在父类，且父类没有初始化，就先初始化父类

<!---->

3.  如果类中存在初始化语句（包括给变量赋值和静态语句块），就依次执行这些初始化语句

注：静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可

以赋值，但是不能访问 

4.  **如果是接口的话︰**

a、初始化一个类的时候，并不会先初始化它实现的接口

b、初始化一个接口时，并不会初始化它的父接口

c、只有当程序首次使用接口里面的变量或者是调用接口方法的时候，才会导致接口初始化

5.  调用Classloader类的loadClass方法来装载一个类，并不会初始化这个类，不是对类的主动使用

**Class.forName和Classloader.loadClass的区别：**

```
Class.forName(className)方法，内部实际调用的方法是  
Class.forName(className,true,classloader);

第2个boolean参数表示类是否需要初始化，  Class.forName(className)默认是需要初始化。
一旦初始化，就会触发目标对象的 static块代码执行，static参数也也会被再次初始化。


ClassLoader.loadClass(className)方法，内部实际调用的方法是  
ClassLoader.loadClass(className,false);

第2个 boolean参数，表示目标对象是否进行链接，false表示不进行链接
由上面介绍可以，不进行链接意味着不进行包括初始化等一些列步骤，那么静态块和静态对象就不会得到执行
```



Java虚拟机必须保证一个类的()方法在多线程环境中被正确地加锁同步，如果多个线程同 时去初始化一个类，那么只会有其中一个线程去执行这个类的()方法，其他线程都需要阻塞等 待，直到活动线程执行完毕()方法。如果在一个类的()方法中有耗时很长的操作，那就 可能造成多个进程阻塞。 需要注意，其他线程虽然会被阻塞，但如果执行＜clinit＞()方法的那条线程退出＜clinit＞()方法 后，其他线程唤醒后则不会再次进入＜clinit＞()方法。同一个类加载器下，一个类型只会被初始化一 次 



### 类的初始化时机

Java程序对类的使用方式分成:主动使用和被动使用，JVM必须在每个类或接口“首次主动使用”时才初始化它们;被动使用类不会导致类的初始化

被动使用的情况：

1.  子类引用父类的静态字段，不会导致子类初始化 child.parentStaticStr
1.  new对象数组也不会初始化该类 ClassA[] aaa = new ClassA[2];

<!---->

3.  访问类的常量也不会初始化该类 A.finalValue

主动使用的情况︰

1.  创建类实例
1.  访问某个类或接口的静态变量

<!---->

3.  调用类的静态方法
3.  反射某个类

<!---->

5.  初始化某个类的子类，而父类还没有初始化
5.  JVM启动的时候运行的主类（main方法在的类）

<!---->

7.  定义了default方法的接口，当接口实现类初始化时

因为不知道实现类是否要重写该方法，若不重写，就需要直接使用接口的方法，所以需要初始化接口




示例，分析该代码：

```
public class MyClassA {
	private static MyClassA myclassA = new MyClassA( );
	private static int a = 0;
	private static int b ;
    private MyclassA( ) {
		a++;
		b++;
		System.out.println("now in MyClassA init( ) a="+a+",b ="+b);
	}
	public static MyClassA getInstance() {
		return myclassA;
	}
    public int getA() {
    	return a;
    }
    public int getB() {
    	return b;
    }
```

```
public static void main (String[] args)
{
    MyClassA classA = MyClassA.getInstance();
    System.out.println(classA.getA());
    System.out.println(classA.getB());
}
```

输出为a=0 b=1

因为：在调用MyClassA的静态方法时，会加载MyClassA，在连接阶段，会给a和b赋默认值0，然后在初始化阶段，为myclassA生成静态对象，调用MyClassA()时，会将a和b都加1，此时都为1，myclassA初始化完成后，初始化a和b，赋初始值，即a=0，而因为b没有赋值操作，故为1

## 类的使用

首先我们需要明确一点，前面**类的加载，除了部分系统类之外，用户定义的类只有在第一次使用的时候才会被加载**

****

### 类的实例化

[类的实例化](https://www.yuque.com/normalgamer/msmcvb/fsy8vg#i0ORj)

#### <clinit>() 与 <init>()区别

**类构造器<clinit>()与实例构造器<init>()** 不同，它不需要程序员进行显式调用，虚拟机会保证在子类类构造器<clinit>()执行之前，父类的类构造<clinit>()执行完毕。\
在一个类的生命周期中，类构造器<clinit>()最多会被虚拟机调用一次，而实例构造器<init>()则会被虚拟机调用多次，只要程序员还在创建对象\
需要注意的是，类的实例化不一定发生在类的初始化完成之后，类初始化的过程中就可能会实例化对象（实际情况下程序员应该尽量避免出现这种情况的，使用为初始化完全的类实例化对象会引起一个意想不到的result......） 

## 类的卸载

当代表一个类的Class对象不再被引用，那么Class对象的生命周期就结束了，对应的在方法区中的数据也会被卸载

Jvm自带的类加载器装载的类，是不会卸载的，由用户自定义的类加载器加载的类是可以卸载的

# 类加载器使用示例

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ba4d154ebca4bcdb784642da90aeb8a~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d29a65f80814041983442e0da454681~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3158b9bc022b4374a63d6b7b9987f4d6~tplv-k3u1fbpfcp-zoom-1.image)

```
import java.sql.Driver;
public class classLoaderStudy {
	public static void main(String[] args) throws Exception {
		String str = "Hello class Loader" ;
		System.out.println("str class loader=="+str.getClass().getClassLoader());
        
    	Class driver = Class.forName("java.sql.Driver" ) ;
		System.out.println("driver class loader="+driver.getClassLoader());
        
        classLoaderStudy t = new ClassLoaderStudy () ;
		System.out.println("'t class loader=="+t.getClass( ).getclassLoader());
}
```

输出：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1549242903e4041aa3d911eba3ea86e~tplv-k3u1fbpfcp-zoom-1.image)

注：String的类加载器为nul，说明是启动类加载器，因为启动类加载器不允许用户获得和使用

```
Classloader可以通过getparent()获取父加载器，比如AppClassloder的父加载器是PlatformClassloder
```

## 类加载器说明

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75497ddd4cdd4178a8658889f429db74~tplv-k3u1fbpfcp-zoom-1.image)




### JDK8的类加载器

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54d42a53a4d14bbd956fe556e9265ec9~tplv-k3u1fbpfcp-zoom-1.image)




# 双亲委派模型

JVM中的ClassLoader通常采用双亲委派模型，要求除了启动类加载器外，其余的类加载器都应该有自己的父级加载器。这里的父子关系是组合而不是继承，工作过程如下︰

1 )一个类加载器接收到类加载请求后，首先搜索它的内建加载器定义的所有“具名模块”

2)如果找到了合适的模块定义，将会使用该加载器来加载

3）如果class没有在这些加载器定义的具名模块中找到，那么将会委托给父级加载器，直到启动类加载器

4）如果父级加载器反馈它不能完成加载请求，比如在它的模块下找不到这个类，那子的类加载器才自己来加载

5）在类路径下找到的类将成为这些加载器的无名模块

即：先找模块，再找classpatth

**而在JDK8中，没有找模块的步骤，而是从classpath中加载，即子类加载器会直接请求父类加载器加载，只有父类加载器都找不到，才让子类加载器从classpath中加载类**

****

**说明：**

1、双亲委派模型对于保证Java程序的稳定运作很重要

因为对于公用的类，会由上级的父类加载器加载，如String类会有启动类加载器加载，这就保证了在程序中，加载到的String类都是同一个类且为java官方的类。也避免了自定义加载器恶意加载自定义同名类，造成安全问题、

2、实现双亲委派的代码在java.lang.ClassLoader的loadClass()方法中，如果自定义类加载器的话，推荐覆盖实现findClass()方法

3、如果有一个类加载器能加载某个类，称为定义类加载器，所有能成功返回该类的Class的类加载器都被称为初始类加载器

4、如果没有指定父加载器，默认就是启动加载器

5、对于任意一个类，都必须由加载它的类加载器和这个类本身一起共同确立其在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名称空间。这句话可以表达得更通俗一些：比较两个类是否“相 等”，只有在这两个类是由同一个类加载器加载的前提下才有意义，否则，即使这两个类来源于同一个 Class文件，被同一个Java虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等。 

举个例子：

```
public class ClassLoaderTest {
	public static void main(String[] args) throws Exception {
		ClassLoader myLoader = new ClassLoader() {
			@Override
			public Class<?> loadClass(String name) throws ClassNotFoundException {
				try {
					String fileName = name.substring(name.lastIndexOf(".") + 1)+".class";
					InputStream is = getClass().getResourceAsStream(fileName);
					if (is == null) {
						return super.loadClass(name);
					}
					byte[] b = new byte[is.available()];
					is.read(b);
					return defineClass(name, b, 0, b.length);
				} catch (IOException e) {
					throw new ClassNotFoundException(name);
				}
			}
		};
		Object obj = myLoader.loadClass("org.fenixsoft.classloading.ClassLoaderTest").newInstance();
		System.out.println(obj.getClass());
		System.out.println(obj instanceof org.fenixsoft.classloading.ClassLoaderTest);
	}
}	
```

结果：

```
class org.fenixsoft.classloading.ClassLoaderTest
false
```

两行输出结果中，从第一行可以看到这个对象确实是类org.fenixsoft.classloading.ClassLoaderTest实 例化出来的，但在第二行的输出中却发现这个对象与类org.fenixsoft.classloading.ClassLoaderTest做所属类型检查的时候返回了false。这是因为Java虚拟机中同时存在了两个ClassLoaderTest类，一个是由虚拟 机的应用程序类加载器所加载的，另外一个是由我们自定义的类加载器加载的，虽然它们都来自同一 个Class文件，但在Java虚拟机中仍然是两个互相独立的类，做对象所属类型检查时的结果自然为 false 

6、运行时包由同一个类加载器的类构成，决定两个类是否属于同一个运行时包，不仅要看全路径名是否一样，还要看定义类加载器是否相同。只有属于同一个运行时包的类才能实现相互包内可见

# 自定义类加载器

```
public class MyClassLoader extends ClassLoader{
	private String myName= "";
	public MyClassLoader( String myName) {
        this.myName = myName;
	}
    
	@Override
	protected Class<?> findclass(String name) throws ClassNotFoundException {
		byte[] data = this.loadclassData(name) ;
		return this.defineclass(name,data，0,data.length);
	}
    
	private byte[] loadclassDatal(String clsName) {
		byte[] data = null;
		InputStream in = null;
		ByteArrayOutputStream out = new ByteArrayOutputStream( ) ;
        clsName = clsName.replace(".","/");
		try(out){
            //classes文件夹下
			in = new FileInputStream( new File("classes/"+clsName+".class"));
            int a = 0;
			while((a = in.read()) != -1) {
				out.write(a);
			}
			data = out.toByteArray( );
        }
        catch(Exception err) {
			err.printStackTrace( );
		}
        return data;
    }
```

测试

```
MyClassLoader myclassLoader = new MyclassLoader( "myClassloder1");
class cls1 = myClassLoader.loadClass("com.cc.jvm.classloader.MyClass");
System.out.println("cls1 class loader =="+cls1.getClassLoader());
```

输出是AppClassloader，因为根据双亲委派模型，且在AppClassloader的类路径下有MyClass类：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70f59ee126a04c7c859324a894503e23~tplv-k3u1fbpfcp-zoom-1.image)

\


删掉bin目录下的MyClass.class文件后，再次启动测试

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b2d697bc83f4248a609416652ef9c9c~tplv-k3u1fbpfcp-zoom-1.image)

\


**自定义类加载器的好处**：

在加载类前后可以实现一些自定义的操作，比如检验等等

# 破坏双亲委派模型

### 第一次破坏

双亲委派模型的第一次“被破坏”其实发生在双亲委派模型出现之前--即JDK1.2发布之前。由于双亲委派模型是在JDK1.2之后才被引入的,而类加载器和抽象类java.lang.ClassLoader则是JDK1.0时候就已经存在，面对已经存在的用户自定义类加载器的实现代码，Java设计者引入双亲委派模型时不得不做出一些妥协。为了向前兼容，JDK1.2之后的java.lang.ClassLoader添加了一个新的proceted方法findClass()，在此之前，用户去继承java.lang.ClassLoader的唯一目的就是重写loadClass()方法，因为虚拟在进行类加载的时候会调用加载器的私有方法loadClassInternal()，而这个方法的唯一逻辑就是去调用自己的loadClass()。JDK1.2之后已不再提倡用户再去覆盖loadClass()方法，应当把自己的类加载逻辑写到findClass()方法中，在loadClass()方法的逻辑里，如果父类加载器加载失败，则会调用自己的findClass()方法来完成加载，这样就可以保证新写出来的类加载器是符合双亲委派模型的。


### 第二次破坏：

双亲委派模型的第二次“被破坏”是由这个模型自身的缺陷所导致的，双亲委派很好地解决了各个类加载器的基础类的同一问题（越基础的类由越上层的加载器进行加载），基础类之所以称为“基础”，是因为它们总是作为被用户代码调用的API，但世事往往没有绝对的完美。

**如果基础类又要调用回用户的代码，那该么办？**

一个典型的例子就是JNDI服务，JNDI现在已经是Java的标准服务，它的代码由启动类加载器去加载（在JDK1.3时放进去的rt.jar），但JNDI的目的就是对资源进行集中管理和查找，它需要调用由独立厂商实现并部署在应用程序的ClassPath下的JNDI接口提供者的代码，但启动类加载器不可能“认识”这些代码。

为了解决这个问题，Java设计团队只好引入了一个不太优雅的设计：**线程上下文类加载器**(Thread Context ClassLoader)。这个类加载器可以通过java.lang.Thread类的setContextClassLoader()方法进行设置，如果创建线程时还未设置，他将会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过的话，那这个类加载器默认就是应用程序类加载器。

有了线程上下文加载器，JNDI服务就可以使用它去加载所需要的SPI代码，也就是父类加载器请求子类加载器去完成类加载的动作，这种行为实际上就是打通了双亲委派模型层次结构来逆向使用类加载器，实际上已经违背了双亲委派模型的一般性原则，但这也是无可奈何的事情。Java中所有涉及SPI的加载动作基本上都采用这种方式，例如JNDI、JDBC、JCE、JAXB和JBI等。

### 第三次破坏：

双亲委派模型的第三次“被破坏”是由于用户对程序动态性的追求导致的，这里所说的“动态性”指的是当前一些非常“热门”的名词：代码热替换、模块热部署等，简答的说就是机器不用重启，只要部署上就能用。

OSGi实现模块化热部署的关键则是它自定义的类加载器机制的实现。每一个程序模块(Bundle)都有一个自己的类加载器，当需要更换一个Bundle时，就把Bundle连同类加载器一起换掉以实现代码的热替换。在OSGi幻境下，类加载器不再是双亲委派模型中的树状结构，而是进一步发展为更加复杂的网状结构，当受到类加载请求时，OSGi将按照下面的顺序进行类搜索：

1）将java.＊开头的类委派给父类加载器加载。

2）否则，将委派列表名单内的类委派给父类加载器加载。

3）否则，将Import列表中的类委派给Export这个类的Bundle的类加载器加载。

4）否则，查找当前Bundle的ClassPath，使用自己的类加载器加载。

5）否则，查找类是否在自己的Fragment Bundle中，如果在，则委派给Fragment Bundle的类加载器加载。

6）否则，查找Dynamic Import列表的Bundle，委派给对应Bundle的类加载器加载。

7）否则，类加载器失败。



### 双亲委派模型破坏举例（JDBC）

在JDBC4.0以后，开始支持使用spi的方式来注册这个Driver，具体做法就是在mysql的jar包中的META-INF/services/java.sql.Driver 文件中指明当前使用的Driver是哪个，然后使用的时候就直接这样就可以了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/323048b72c8f4c66a22ebedda0d6c75c~tplv-k3u1fbpfcp-zoom-1.image)

```
 Connection conn= DriverManager.getConnection("jdbc:mysql://localhost:3306/test?characterEncoding=GBK", "root", "");
```

现在，我们分析下看使用了这种spi服务的模式原本的过程是怎样的:

1.  从META-INF/services/java.sql.Driver文件中获取具体的实现类名“com.mysql.cj.jdbc.Driver”
1.  加载这个类，用class.forName(“com.mysql.jdbc.Driver”)来加载

Class.forName()加载用的是调用者的Classloader, 这个调用者DriverManager是在rt.jar中的，ClassLoader是启动类加载器，而com.mysql.jdbc.Driver肯定不在<JAVA_HOME>/lib下，所以肯定是无法加载mysql中的这个类的。这就是双亲委派模型的局限性了，父级加载器无法加载子级类加载器路径中的类。那么JDK是什么做的呢？

```
public class DriverManager {
    static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }
    private static void loadInitialDrivers() {
        //省略代码
        //这里就是查找各个sql厂商在自己的jar包中通过spi注册的驱动
        ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
        Iterator<Driver> driversIterator = loadedDrivers.iterator();
        try{
             while(driversIterator.hasNext()) {
                driversIterator.next();
             }
        } catch(Throwable t) {
                // Do nothing
        }

        //省略代码
    }
}
```

其中的ServiceLoader.load(Driver.class)

```
   public static <S> ServiceLoader<S> load(Class<S> service) {
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
    }
```

获取线程上下文类加载器Thread.currentThread().getContextClassLoader(); 这个值如果没有特定设置,一般默认使用的是应用程序类加载器;

#### 如何不破坏双亲委派模型加载数据库驱动：

```
 // 1.加载数据访问驱动
Class.forName("com.mysql.cj.jdbc.Driver");
//2.连接到数据"库"上去
Connection conn= DriverManager.getConnection("jdbc:mysql://localhost:3306/test?characterEncoding=GBK", "root", "");
```

这样就行

Class.forName("com.mysql.cj.jdbc.Driver"); 这句会主动去加载类com.mysql.cj.jdbc.Driver

```
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    //
    // Register ourselves with the DriverManager
    //
    static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    public Driver() throws SQLException {
        // Required for Class.forName().newInstance()
    }
}
```

可以看到，Class.forName()其实触发了静态代码块，然后向DriverManager中注册了一个mysql的Driver实现。这个时候，我们通过DriverManager去获取connection的时候只要遍历当前所有Driver实现，然后选择一个建立连接就可以了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff1f17b9204549e3880fbd5d26abc9de~tplv-k3u1fbpfcp-zoom-1.image)

