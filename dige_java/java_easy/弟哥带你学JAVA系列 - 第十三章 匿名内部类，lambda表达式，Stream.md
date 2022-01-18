## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 匿名内部类

<https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html>

匿名内部类可以使你的代码更加简洁，**你可以在定义一个类的同时对其进行实例化**。它与局部类很相似，不同的是**它没有类名**，**如果某个局部类你只需要用一次，那么你就可以使用匿名内部类**

****

## 定义匿名内部类

##

```
public class HelloWorldAnonymousClasses {

    /**
     * 包含两个方法的HelloWorld接口
     */
    interface HelloWorld {
        public void greet();
        public void greetSomeone(String someone);
    }

    public void sayHello() {

        // 1、局部类EnglishGreeting实现了HelloWorld接口
        class EnglishGreeting implements HelloWorld {
            String name = "world";
            public void greet() {
                greetSomeone("world");
            }
            public void greetSomeone(String someone) {
                name = someone;
                System.out.println("Hello " + name);
            }
        }

        HelloWorld englishGreeting = new EnglishGreeting();

        // 2、匿名类实现HelloWorld接口
        HelloWorld frenchGreeting = new HelloWorld() {
            String name = "tout le monde";
            public void greet() {
                greetSomeone("tout le monde");
            }
            public void greetSomeone(String someone) {
                name = someone;
                System.out.println("Salut " + name);
            }
        };


        englishGreeting.greet();
        frenchGreeting.greetSomeone("Fred");
        
   }

    public static void main(String... args) {
        HelloWorldAnonymousClasses myApp = new HelloWorldAnonymousClasses();
        myApp.sayHello();
        
        /** 输出：
        Hello world
        Salut Fred
        **/
    }
}
```

该例中用局部类来初始化变量englishGreeting，用匿类来初始化变量frenchGreeting和spanishGreeting，两种实现之间有明显的区别：

1）局部类EnglishGreetin继承HelloWorld接口，有自己的类名，定义完成之后需要再用new关键字实例化才可以使用；

2）frenchGreeting在定义的时候就实例化了，定义完了就可以直接使用；

3）匿名类是一个表达式，因此在定义的最后用分号";"结束。

## 匿名内部类的语法

如上文所述，匿名类是一个表达式，**匿名类的语法就类似于调用一个类的构建函数（new HelloWorld()），除些之外，还包含了一个代码块，在代码块中完成类的定义**。除了上面的实现接口的例子之外，还可以继承父类：

```
public class AnimalTest {

    private final String ANIMAL = "动物";

    public void accessTest() {
        System.out.println("匿名内部类访问其外部类方法");
    }

    class Animal {
        private String name;

        public Animal(String name) {
            this.name = name;
        }

        public void printAnimalName() {
            System.out.println(bird.name);
        }
    }

    // 鸟类，匿名子类，继承自Animal类，可以覆写父类方法
    Animal bird = new Animal("布谷鸟") {

        @Override
        public void printAnimalName() {
            accessTest();   　　　　　　　　// 访问外部类成员
            System.out.println(ANIMAL);  // 访问外部类final修饰的变量
            super.printAnimalName();
        }
    };

    public void print() {
        bird.printAnimalName();
    }

    public static void main(String[] args) {

        AnimalTest animalTest = new AnimalTest();
        animalTest.print();
    }
}
```

运行结果：

```
匿名内部类访问其外部类方法
动物
布谷鸟
```

从以上两个实例中可知，匿名类表达式包含以下内部分：

-   操作符：new；
-   一个要实现的接口或要继承的类，案例一中的匿名类实现了HellowWorld接口，案例二中的匿名内部类继承了Animal父类；

<!---->

-   一对括号，如果是匿名子类，与实例化普通类的语法类似，如果有构造参数，要带上构造参数；如果是实现一个接口，只需要一对空括号即可；
-   一段被"{}"括起来类声明主体；

<!---->

-   末尾的";"号（因为匿名类的声明是一个表达式，是语句的一部分，因此要以分号结尾）。

## 访问作用域的局部变量、定义和访问匿名内部类成员

匿名内部类与局部类对作用域内的变量拥有相同的的访问权限。

(1)、匿名内部类可以访问外部类的所有成员；

(2)、匿名内部类不能访问外部类未加final修饰的变量（注意：JDK1.8即使没有用final修饰也可以访问）；

(3)、属性屏蔽，与内嵌类相同，匿名内部类定义的类型（如变量）会屏蔽其作用域范围内的其他同名类型（变量）：

(4)、匿名内部类中不能定义静态属性、方法； 

(5)、匿名内部类可以有常量属性（final修饰的属性）；

(6)、匿名内部内中可以定义属性，如上面代码中的代码:private int x = 1;

(7)、匿名内部内中可以可以有额外的方法（父接口、类中没有的方法）;

(8)、匿名内部内中可以定义内部类；

(9)、匿名内部内中可以对其他类进行实例化。

# Lambda表达式




## 函数式接口：

函数接口是只有一个方法的接口，用作lambda表达式的类型

例子：

```
public class javatest {
    
       interface h1
       {
        void print();
       }

       h1 h = () -> System.out.println("aaaaaa");

    public static void main(String[] args) {

         javatest j = new javatest();
          j.h.print();
    }
}
```

输出：

```
aaaaaa
```

-   Lambda 表达式主要用来定义行内执行的方法类型接口，例如，一个简单方法接口。在上面例子中，我们使用各种类型的Lambda表达式来定义MathOperation接口的方法。然后我们定义了sayMessage的执行。
-   Lambda 表达式免去了使用匿名方法的麻烦，并且给予Java简单但是强大的函数化的编程能力。

## 变量作用域

lambda 表达式**只能引用标记了 final 的外层局部变量**，这就是说不能在 lambda 内部修改定义在域外的局部变量，否则会编译错误。

```
public class javatest {


    final static String salutation = "Hello! ";

    public static void main(String args[]){
        GreetingService greetService1 = message ->
                System.out.println(salutation + message);
        greetService1.sayMessage("gangster");
    }

    interface GreetingService {
        void sayMessage(String message);
    }
}
```

输出

```
Hello! gangster
```

在 Lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量。

```
public class javatest {

    public static void main(String args[]) {
      final int num = 1;
        
       Converter<Integer, Integer> s = (param,num) -> System.out.println((param + num));
        s.convert(2,8);
        
        //Error:(13, 47) java: 已在方法 main(java.lang.String[])中定义了变量 num
    }

    public interface Converter<T1, T2> {
        void convert(int i,int y);
    }
}
```

若有收获，就点个赞吧

