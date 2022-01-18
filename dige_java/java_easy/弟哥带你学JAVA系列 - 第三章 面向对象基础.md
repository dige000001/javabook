## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂
# 基础知识

1.  类内定义的成员变量会自动初始化（将内存里的比特全设为0），比如一个类定义了static int age; 在main里print(age)时会输出0；
1.  类内的成员变量在类的范围内生效

<!---->

3.  调用函数时，会在栈内给形参分配位置，然后将实参的值赋给形参，函数结束后，栈内的形参被回收
3.  构造函数不能有返回值和修饰符(public、final、static等)

<!---->

5.  new一个对象时，在栈内生成引用（指针），在堆内生成对象




## 重载

1.  方法名一致，形参类型一致会冲突，不算重载
1.  方法名一致，形参类型不一致才是重载

<!---->

3.  由于在java中，整数常量会默认为int类型，所以下图的t.max()调用的是第一个

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b53f8174a7c84977ad9e8a393c81ffde~tplv-k3u1fbpfcp-zoom-1.image)




## static

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4b3cc83d2d44a00993f7e776a2dd222~tplv-k3u1fbpfcp-zoom-1.image)

<https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5>

jvm各数据存储区

# 继承

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50e1aa1b0e544b74b97491b595a73deb~tplv-k3u1fbpfcp-zoom-1.image)

类内部：只允许类内的方法访问该变量

继承的时候，private修饰的变量也会被继承，但无法通过子类内部的方法访问该集成的private变量，只能用父类的方法访问

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44319fb9d80a4ab3b52c5f00382560a1~tplv-k3u1fbpfcp-zoom-1.image)

java中，子类可以定义和父类一样的成员变量，jvm会为两个变量都分配内存空间，但若子类想访问父类的该变量，需要用super关键字




## 重写

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed5359286b864d1a93e623a7792fd536~tplv-k3u1fbpfcp-zoom-1.image)




## 子类的构造方法

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/898b68d930294e3b8417336c07d22c65~tplv-k3u1fbpfcp-zoom-1.image)

# Object类

object类是所有类的根类

## toString 方法

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de6797960bc84d78ae2366c02ec308a4~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b13c35700ae0422fa2a19243ef5537ed~tplv-k3u1fbpfcp-zoom-1.image)

返回 类名@hashcode

## equals方法

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab6fdbde026b4caf854956dbdb39fd1b~tplv-k3u1fbpfcp-zoom-1.image)

equals默认情况比较两个对象地址是否相同（与==一样），故一般来说需要重写该方法

比如String类，就对该方法进行了重写，只要内容相同，就返回true

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f7f46106a574ebe8d7ce19f83bc5032~tplv-k3u1fbpfcp-zoom-1.image)




# 转型

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a99525e209684799b11b673d4947234b~tplv-k3u1fbpfcp-zoom-1.image)

注：Animal a = new Dog()

a instanceof Dog 返回的是true

此时，若想访问a所指的对象的Dog部分，需要：

Dog b = (Dog) a；

# 多态



## 动态绑定




在java中，用引用调用对象的方法时，是根据实际对象调用方法的，比如：

当子类重写父类的方法时，如Son类重写了Father类的方法，那么下面的代码将调用的是Son重写的代码

Father a = new Son();

a.name();

注意：动态绑定只对方法有效，不对成员变量有效！即若Son类里有和Father一样的成员变量，调用a.xxx时返回的是Father类的值！

深入解读：<https://blog.csdn.net/sted_zxz/article/details/77980124>

## 抽象类

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9543b7fd7a2f4720ab301e8ef6f234be~tplv-k3u1fbpfcp-zoom-1.image)

抽象方法必须被重写

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa74d898f0ae4a7d89302b3cd7a9a1dc~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25a4086606bc4dcbbb2b403deae21895~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ef542ebe7a1489e89b35b1240036cb5~tplv-k3u1fbpfcp-zoom-1.image)



#### 接口![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bcb61878a7c45eda18a4dc91a7e7d13~tplv-k3u1fbpfcp-zoom-1.image)

JDK1.8之后，接口可以有静态方法，但必须有方法体，即body

静态方法无需也不能重写

####

####

#### ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d78d7c59f3b04dcb9529476bd284eeea~tplv-k3u1fbpfcp-zoom-1.image)




# 可变参数列表

```
public static void executebindParam(PreparedStatement pstmt,Object ...os)
```

Object ...os这种写法是从Java 5开始的，Java语言对方法参数支持一种新写法，叫可变长度参数列表。

**表示此处接受的参数为0到多个Object类型的对象，或者是一个Object[]**

注意可变长度参数列表的格式：

1.  参数类型和“...”三个点之间不必须有一个空格（Object ...os），Object...os也不会报错误；
1.  可变长度参数列表这个参数必须是参数列表中的最后一个参数，不然会报错

