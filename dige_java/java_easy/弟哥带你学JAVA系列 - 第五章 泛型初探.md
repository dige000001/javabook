## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂


# 泛型类

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b326dc3e70a4130a32a3d4295d9a753~tplv-k3u1fbpfcp-zoom-1.image)

若要调用，只需要

Pair<String> p = new Pair<>();

# 泛型方法

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93ff4ee5cf9d49b396cdc12df1d6cdfe~tplv-k3u1fbpfcp-zoom-1.image)




几乎在大多数情况下，对于泛型方法的类型引用没有问题。偶尔，编译器也会提示错误,此时需要解译错误报告。看一看下面这个示例:

double middle = ArrayAlg.getMiddle(3.14，1729，0);

错误消息会以晦涩的方式指出(不同的编译器给出的错误消息可能有所不同):解释这句代码有两种方法，而且这两种方法都是合法的。简单地说，编译器将会自动打包参数为1个Double和2个Integer对象，而后寻找这些类的共同超类型。事实上;找到2个这样的超类型:Number和 Comparable接口，其本身也是一个泛型类型。在这种情况下，可以采取的补救措施是把所有参数写成Double型

# 类型变量的限定

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5036cd908c7c4538bf8a02b6ce366cf2~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf0905128aa94f34a6851dd08be1ca48~tplv-k3u1fbpfcp-zoom-1.image)

# 泛型代码和虚拟机

Java语言的泛型采用的是擦除法实现的伪泛型，字节码（Code属性）中所有的泛型信息编译（类型变量、参数化类型）在编译之后都通通被擦除掉。使用擦除法的好处是实现简单（主要修改 Javac编译器，虚拟机内部只做了很少的改动）、非常容易实现Backport，运行期也能够节省一些类型 所占的内存空间。但坏处是运行期就无法像C#等有真泛型支持的语言那样，将泛型类型与用户定义的 普通类型同等对待，例如运行期做反射时无法获得泛型信息。Signature属性就是为了弥补这个缺陷而 增设的，现在Java的反射API能够获取的泛型类型，最终的数据来源也是这个属性

Signature属性在JDK 5增加到Class文件规范之中，它是一个可选的定长属性，可以出现于类、字段 表和方法表结构的属性表中。在JDK 5里面大幅增强了Java语言的语法，在此之后，任何类、接口、初 始化方法或成员的泛型签名如果包含了类型变量（Type Variable）或参数化类型（Parameterized Type），则Signature属性会为它记录泛型签名信息 

## 类型擦除




### 无限定类型的擦除

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77fef5199b9846618594ae604eb0b5f4~tplv-k3u1fbpfcp-zoom-1.image)




### 有限定类型的擦除

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/86c06976672f43e4ba115d1dbd01a8c5~tplv-k3u1fbpfcp-zoom-1.image)

即：泛型类在编译时，没有限定类型的话会使用Object类代替，有的话用第一个限定类型代替

## 翻译

### 翻译泛型表达式

当程序调用泛型方法时，如果擦除返回类型，编译器插入强制类型转换。例如，下面这个语句序列

Pair<Employee> buddies = . . .;

Employee buddy = buddies.getFirst();

擦除getFirst的返回类型后将返回Object类型。编译器自动插入 Employee 的强制类型转换。也就是说，编译器把这个方法调用翻译为两条虚拟机指令:

·对原始方法Pair.getFirst的调用。

·将返回的Object类型强制转换为Employee类型。

### 翻译泛型方法

类型擦除也会出现在泛型方法中。程序员通常认为下述的泛型方法

public static <T extends Comparable> T min ( T[] a )

是一个完整的方法族,而擦除类型之后，只剩下一个方法:

public static Comparable min(Comparable[] a)

注意,类型参数T已经被擦除了，只留下了限定类型Comparable。

方法的擦除带来了两个复杂问题。看一看下面这个示例

```
class DateInterval extends Pair<LocalDate>{
	public void setSecond(LocalDate second){
		if (second.compareTo(getFirst())>= 0)
			super.setSecond(second);
}
```

一个DateInterval是一对 LocaIDate对象，并且需要覆盖这个方法来确保第二个值永远不小于第一个值。这个类擦除后变成

```
class DateInterval extends Pair{ // after erasure
	public void setSecond(LocalDate second) { . . .}
}
```

但是除此之外，还有另外一个setSecond方法 public void setSecond(Object second) ，这是从pair继承下来的

考虑一下代码

```
DateInterval interval = new DateInterval(. ..);
Pair<LocalDate> pair = interval; // 将interval引用赋给父类
pair.setSecond(aDate);
```

这里，希望对setSecond的调用具有多态性，并调用最合适的那个方法。由于pair引用DateInterval对象，所以应该调用DateInterval.setSecond。问题在于类型擦除与多态发生了冲突（即该调用DateInterval的哪个setSecond()？）。编译器使用桥方法(bridge method)解决这个问题：

变量pair已经声明为类型Pair<LocalDate>，并且这个类型只有一个简单的方法叫setSecond，即 setSecond(Object)。虚拟机用pair引用的对象调用这个方法。这个对象是DateInterval类型的，因而将会调用DateInterval.setSecond(Object)方法。**这个方法是合成的桥方法，它会调用 DateInterval.setSecond(Date)** ，这正是我们所期望的操作效果。

# 约束和局限性

#### 不能用基本类型实例化类型参数

不能用类型参数代替基本类型。因此，没有Pair<double>，只有Pair<Double>。当然，其原因是类型擦除。擦除之后，Pair类含有Object类型的域，而Object 不能存储double值。

#### 运行时类型查询只适用于原始类型

虚拟机中的对象总有一个特定的非泛型类型。因此，所有的类型查询只产生原始类型。例如:

Pair<String> p = new Pair<>("sss","ssss");

System.*out*.println(p instanceof Pair<String>);//Error

实际上仅仅测试p是否是任意类型的一个Pair。下面的测试同样如此:

System.*out*.println(p instanceof Pair<T>);//Error

同样的道理,getClass方法总是返回原始类型。例如:

```
Pair<String> stringPair = . . .;
Pair<Employee> employeePair = .. ;
if (stringPair.getClass() == employeePair.getClass() // they are equal
```

其比较的结果是true，这是因为两次调用getClass都将返回Pair.classo

#### 不能创建参数化类型的数组

数组存储检查是很严格的，它只能存储**创建时元素类型**

```
Pair[] table = new Pair[10];
table[0] = new Object(); //编译错误
```

很明显直接存储，编译器会报错。那么将数组向上转换一下，再存储呢？

```
Pair[] table = new Pair[10];
Object[] o = table; //自动转换
o[0] = new Object();
```

此时编译器是不会报错的，但是运行时会抛出**ArrayStoreException**异常

但是，泛型擦除会破坏这种机制，举例：

```
Pair<String>[] table = new Pair<String>[10] //假设可以
Object[] o = table; //泛型擦除变为Pair[]，向上自动转换为Object[]
o[0] = new Pair<Double>();
```

**你会发现，此时Pair<String>数组里存储了Pair<Double>元素**

总之java不允许创建参数化的泛型数组，是为了保护数组安全性。

**提示:如果需要收集参数化类型对象，只有一种安全而有效的方法:使用ArrayList:**

ArrayList<Pair<String,String>> a = new ArrayList<>();

千万别写出下面的代码

```
    public static void main(String[] args) {
        Pair<Integer,Integer>[] table = new Pair[10];
        Object[] o =table;
        o[0] = new Pair<Double,Double>(1.2,2.2);
        System.out.println(table[0].equals(o[0]));// true
    }
```

#### 不能实例化类型变量

不能使用像new T(..)，new T[..]或T.class这样的表达式中的类型变量。例如，下面的Pair<T>构造器就是非法的:

public Pair( ){ first = new T(); second = new T(); }// Error

#### 不能在静态域或方法中引用泛型类型

```
public class Singleton<T>{
      private static T singleInstance;  //ERROR

      public static T getSingleInstance(){ //ERROR
          if(singleInstance == null) 
              return singleInstance;
      }
}
```

类擦除后：

```
public class Singleton{
      private static Object singleInstance; 

      public static Obejct getSingleInstance(){
          if(singleInstance == null) 
              return singleInstance;
      }
}
```

当调用此静态方法时，无需创建出这个类的一个实例，语句应该为

```
AType a = Singleton.getSingleInstance());
```

相当于将Object对象赋值给a，很明显，子类变量不允许引用父类对象，必须要有强制类型转换，而在普通的类型擦除后也正是如此，但在这里getSingleInstance()不知道应该返回什么类型，所以这种用法是不允许的。

反过来如果singleInstance和getSingleInstance不是静态的话，代码将如下所示：

```
Singleton<AType> s = new Singleton<AType>();
AType a = s.getSingleInstance();
```

这样是可以的

# 泛型的继承关系

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b8a1381884394a96a018ec78260b6b69~tplv-k3u1fbpfcp-zoom-1.image)

可以看到：ArrayList<Manager>是List<Manager>的子类

**但List<Manager>不是List<Employee>的子类**

# 通配符

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d0f8d354ac84c8090ef403c687f69a5~tplv-k3u1fbpfcp-zoom-1.image)

extends通配符不允许set方法，即使传入的是父类

