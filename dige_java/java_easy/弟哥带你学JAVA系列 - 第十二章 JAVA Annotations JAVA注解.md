## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 基础知识
<https://docs.oracle.com/javase/tutorial/java/annotations/index.html>

<https://blog.csdn.net/qq1404510094/article/details/80577555>

<https://www.jianshu.com/p/9471d6bcf4cf>

注解用处：

Annotations have a number of uses, among them:

-   **Information for the compiler** — Annotations can be used by the compiler to detect errors or suppress warnings.
-   **Compile-time and deployment-time processing** — Software tools can process annotation information to generate code, XML files, and so forth.

<!---->

-   **Runtime processing** — Some annotations are available to be examined at runtime.

1.  **SOURCE 标记一些信息，为编译器提供辅助信息。**




可以为编译器提供而外信息，以便于检测错误，抑制警告等，譬如@Override、@SuppressWarnings等这类注解就是用于标识，可以用作一些检验。

2.  **CLASS 编译时动态处理。**

一般这类注解会在编译的时候，根据注解标识，动态生成一些类或者生成一些xml都可以，在运行时期，这类注解是没有的，也就是在类加载的时候丢弃。

会依靠动态生成的类做一些操作，因为没有反射，效率和直接调用方法没什么区别。ParcelableGenerator、butterknife 、androidannotaion都使用了类似技术

3.  **RUNTIME 运行时动态处理。**

这个大家见得应该最多，在运行时拿到类的Class对象，然后遍历其方法、变量，判断有无注解声明，然后做一些事情。

譬如使用表单验证注解@Validate，不保留活动的@SaveInstance




### 注解的定义

注解其实就是一种特殊的接口，定义方式：

public @interface MyAnnotation {

int attribute1 () default 1;

String attribute2 () default "wuzhijun";

……

}

### 注解的使用

@MyAnnotation(attribute=2)

public class Myclass{……}

没有属性的注解可以不加属性值

### Java内置的注解

1.  **@Deprecated**

这个元素是用来标记过时的元素，想必大家在日常开发中经常碰到。编译器在编译阶段遇到这个注解时会发出提醒警告，告诉开发者正在调用一个过时的元素比如过时的方法、过时的类、过时的成员变量。

2.  **@Override**

提示子类要复写父类中被 @Override 修饰的方法

3.  **@SupressWarnings**

阻止警告的意思，如 @SupressWarnings("deprecated")

4.  **SafeVarargs**

参数安全类型注解。它的目的是提醒开发者不要用参数做一些不安全的操作,它的存在会阻止编译器产生 unchecked 这样的警告。

5.  **@FunctionalInterface**

函数式接口注解，这个是 Java 1.8 版本引入的新特性。函数式编程很火，所以 Java 8 也及时添加了这个特性。函数式接口 (Functional Interface) 就是一个具有一个方法的普通接口。

### 元注解

元注解顾名思义我们可以理解为注解的注解，它是作用在注解中，方便我们使用注解实现想要的功能。元注解分别有@Retention、 @Target、 @Document、 @Inherited和@Repeatable（JDK1.8加入）五种。

1.  **@Retention**

-   Retention英文意思有保留、保持的意思，它表示注解存在阶段是保留在源码（编译期），字节码（类加载）或者运行期（JVM中运行）。在@Retention注解中使用枚举RetentionPolicy来表示注解保留时期
-   @Retention(RetentionPolicy.SOURCE)，注解仅存在于源码中，在class字节码文件中不包含

<!---->

-   @Retention(RetentionPolicy.CLASS)， 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得
-   @Retention(RetentionPolicy.RUNTIME)， 注解会在class字节码文件中存在，在运行时可以通过反射获取到

<!---->

-   如果我们是自定义注解，则通过前面分析，我们自定义注解如果只存着源码中或者字节码文件中就无法发挥作用，而在运行期间能获取到注解才能实现我们目的，所以自定义注解中肯定是使用  **@Retention(RetentionPolicy.RUNTIME)**

2.  **@Target**

-   Target的英文意思是目标，这也很容易理解，使用@Target元注解表示我们的注解作用的范围就比较具体了，可以是类，方法，方法参数变量等，同样也是通过枚举类ElementType表达作用类型
-   @Target(ElementType.TYPE) 作用接口、类、枚举、注解

<!---->

-   @Target(ElementType.FIELD) 作用属性字段、枚举的常量
-   @Target(ElementType.METHOD) 作用方法

<!---->

-   @Target(ElementType.PARAMETER) 作用方法参数
-   @Target(ElementType.CONSTRUCTOR) 作用构造函数

<!---->

-   @Target(ElementType.LOCAL_VARIABLE)作用局部变量
-   @Target(ElementType.ANNOTATION_TYPE)作用于注解（@Retention注解中就使用该属性）

<!---->

-   @Target(ElementType.PACKAGE) 作用于包
-   @Target(ElementType.TYPE_PARAMETER) 作用于类型泛型，即泛型方法、泛型类、泛型接口 （jdk1.8加入）

<!---->

-   @Target(ElementType.TYPE_USE) 类型使用.可以用于标注任意类型除了 class （jdk1.8加入）
-   **一般比较常用的是ElementType.TYPE类型**

3.  **@Document**

-   Document的英文意思是文档。它的作用是能够将注解中的元素包含到 Javadoc 中去。

4.  **@Inherited**

-   Inherited的英文意思是继承，但是这个继承和我们平时理解的继承大同小异，一个被@Inherited注解了的注解修饰了一个父类，如果他的子类没有被其他注解修饰，则它的子类也继承了父类的注解。

5.  **@Repeatable**

-   Repeatable的英文意思是可重复的。顾名思义说明被这个元注解修饰的注解可以同时作用一个对象多次，但是每次作用注解又可以代表不同的含义。

