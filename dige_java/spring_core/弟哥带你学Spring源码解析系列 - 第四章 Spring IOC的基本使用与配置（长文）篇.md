## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

## 1. 基于XML方式

### < beans >和< bean >

所有使用 XML 文 件进行配置信息加载的Spring IoC 容器，包括 BeanFactory 和 ApplicationContext的所有XML相应实现，都使用统一的XML格式。 在Spring 2.0版本之前，这种格 式由Spring提供的DTD规定，也就是说，所有的Spring容器加载的XML配置文件的头部，都需要以下 形式的DOCTYPE声明： 

```
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" ➥
"http://www.springframework.org/dtd/spring-beans.dtd"> 
<beans> 
 ... 
</beans> 
```

Spring 2.0版本之后，Spring在继续保持向前兼容的前提下，既可以继续使用DTD方式的DOCTYPE 进行配置文件格式的限定，又引入了基于XML Schema的文档声明。所以，Spring 2.0之后，同样可以 使用代码清单4-11所展示的基于XSD的文档声明。 

```
<beans xmlns="http://www.springframework.org/schema/beans" 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
xmlns:util="http://www.springframework.org/schema/util" 
xmlns:jee="http://www.springframework.org/schema/jee" 
xmlns:lang="http://www.springframework.org/schema/lang" 
xmlns:aop="http://www.springframework.org/schema/aop" 
xmlns:tx="http://www.springframework.org/schema/tx" 
xsi:schemaLocation="http://www.springframework.org/schema/beans 
http://www.springframework.org/schema/beans/spring-beans-2.0.xsd 
http://www.springframework.org/schema/util 
http://www.springframework.org/schema/util/spring-util-2.0.xsd 
http://www.springframework.org/schema/jee 
http://www.springframework.org/schema/jee/spring-jee-2.0.xsd 
http://www.springframework.org/schema/lang 
http://www.springframework.org/schema/lang/spring-lang-2.0.xsd 
http://www.springframework.org/schema/aop 
http://www.springframework.org/schema/aop/spring-aop-2.0.xsd 
http://www.springframework.org/schema/tx 
http://www.springframework.org/schema/tx/spring-tx-2.0.xsd"> 
</beans> 
```

#### <beans>详解

<beans>是XML配置文件中最顶层的元素，它下面可以包含0或者1个和多个 以及或者，如图所示。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e3e4cfa326040ff8999a15502cd920a~tplv-k3u1fbpfcp-zoom-1.image)

<beans>作为所有的“统帅”，**它拥有相应的属性（attribute）对所辖的<bean>进行统一 的默认行为设置**，包括如下几个。 

**default-lazy-init：** 其值可以指定为true或者false，默认值为false。用来标志是否对所 有的进行延迟初始化。  **default-autowire：** 可以取值为no、byName、byType、constructor以及autodetect。默 认值为no，如果使用自动绑定的话，用来标志全体bean使用哪一种默认绑定方式。  

**default-dependency-check：** 可以取值none、objects、simple以及all，默认值为none， 即不做依赖检查。  **default-init-method：** 如果所管辖的按照某种规则，都有同样名称的初始化方法的 话，可以在这里统一指定这个初始化方法名，而不用在每一个上都重复单独指定。  

**default-destroy-method**：与default-init-method相对应，如果所管辖的bean有按照某种 规则使用了相同名称的对象销毁方法，可以通过这个属性

#### <description>、<import>和<alias>

**<description>：** 可以通过<description>在配置的文件中指定一些描述性的信息。

**<import>：** 通常情况下，可以根据模块功能或者层次关系，将配置信息分门别类地放到多个配置文件中。在 想加载主要配置文件，并将主要配置文件所依赖的配置文件同时加载时，可以在这个主要的配置文件 中通过元素对其所依赖的配置文件进行引用。比如，如果A.xml中的定义可能依赖 B.xml中的某些定义，那么就可以在A.xml中使用将B.xml引入到A.xml，以类似于 的形式。 

**<alias>：** 可以通过<alias>为某些<bean>起一些“外号”（别名），通常情况下是为了减少输入。比如， 假设有个<bean>，它的名称为dataSourceForMasterDatabase，你可以为其添加一个<alias>，像 这样<alias name="dataSourceForMasterDatabase" alias="masterDataSource"/>。以后通过 dataSourceForMasterDatabase或者masterDataSource来引用这个<bean>都可以，只要你觉得方便 就行。

### 创建对象

```
<bean id="user" class="com.wu.spring.User"></bean>
```

（1）在 spring 配置文件中，使用 bean 标签，标签里面添加对应属性，就可以实现对象创建 

（2）在 bean 标签有很多属性，介绍常用的属性

-   id 属性：唯一标识 
-   class 属性：类全路径（包类路径）

<!---->

-   name：可以用来指定别名， 还可以通过逗号、空格或者冒号分割指定多个name 。
-   layz-init：是否延迟初始化

（3）创建对象时候，默认也是执行无参数构造方法完成对象创建 

（4）有参构造：（需要在类的定义中定义有参构造方法）

```
<bean id="orders" class="com.atguigu.spring5.Orders" name="myOders，simpleOrders" lazy-init="true">
 <constructor-arg name="oname" value="电脑"></constructor-arg>
 <constructor-arg name="address" value="China"></constructor-arg>
</bean>
```

若某类声明了两个构造方法，分别都只是传入一个参数，且参数类型不同。这时，我们可以进行配置，通过指定构造方法的参数类型来解决这一问题 ，如以下代码所示： 

```
<bean id="mockBO" class="..MockBusinessObject"> 
 <constructor-arg type="int"> 
 <value>111111</value> 
 </constructor-arg> 
</bean> 
```

如果某些时候，我们没有通过类似<ref>的元素明确指定对象A依赖于对象B的话，如何让容器在实例化对象A之前首先实例化对象B呢？考虑以下所示代码：

```
public class SystemConfigurationSetup 
{ 
 static 
 { 
 DOMConfigurator.configure("配置文件路径"); 
 // 其他初始化代码
 } 
} 
```

系统中所有需要日志记录的类，都需要在这些类使用之前首先初始化log4j。那么，就会非显式地依赖于SystemConfigurationSetup的静态初始化块。如果ClassA需要使用log4j，那么就必须在bean 定义中使用depends-on来要求容器在初始化自身实例之前首先实例化SystemConfigurationSetup， 以保证日志系统的可用，如下代码演示的正是这种情况： 

```
<bean id="classAInstance" class="...ClassA" depends-on="configSetup"/> 
<bean id="configSetup" class="SystemConfigurationSetup"/> 
```

### 注入属性

（1）使用setter方法注入基本类型属性值：（需要在类的定义中定义好setter属性）

```
<bean id="book" class="com.atguigu.spring5.Book">
 <!--使用 property 完成属性注入
 name：类里面属性名称 如 String bname;
 value：向属性注入的值，传入的是String类型，IoC容器会帮忙进行类型转换
 -->
 <property name="bname" value="易筋经"></property>
 <property name="bauthor" value="达摩老祖"></property>
 <property name="byear">
   <!--注入null 值-->
 <null/>
 </property>
</bean>
```

（2）注入对象

（外部Bean）

```
<!--1 service 和 dao 对象创建-->
<bean id="userService" class="com.atguigu.spring5.service.UserService">
 <!--注入 userDao 对象
 UserService类的定义里有对UserDao的依赖 UserDao userDao;
 ref的值要与外部Bean的id一致
 -->
 <property name="userDao" ref="userDaoImpl"></property>
  <!--级联赋值 要求UserDaoImpl有getter和setter方法-->
   <property name="userDao.name" value="wu"></property>
</bean>

<!--创建UserDaoImpl-->
<bean id="userDaoImpl" class="com.atguigu.spring5.dao.UserDaoImpl">
  <property name="userDao.name" value="li"></property>
</bean>
```

（内部Bean）
```

<bean id="emp" class="com.atguigu.spring5.bean.Emp">
 <!--这样就保证了dept这个Bean只注入到emp，其他Bean无法引用到dept-->
 <property name="dept">
 		<bean id="dept" class="com.atguigu.spring5.bean.Dept">
 				<property name="dname" value="安保部"></property>
 		</bean>
 </property>
</bean>
```

### 注入集合属性

```
 <!--set 类型属性注入-->
 <property name="sets">
	 <set>
 				<value>MySQL</value>
 				<value>Redis</value>
 	 </set>
</property>

<!--map 类型属性注入-->
 <property name="maps">
 		<map>
 				<entry key="JAVA" value="java"></entry>
 				<entry key="PHP" value="php"></entry>
	 </map>
 </property>

<!--注入 list 集合类型，值是对象-->
<property name="courseList">
	  <list>
				 <ref bean="course1"></ref>
				 <ref bean="course2"></ref>
	  </list>
</property>
```

通过utils注入集合：看PDF笔记

### lookup-method

lookup-method 似乎并不是很常用，但是在某些时候它的确是非常有用的属性，通常我们称它为获取器注入。引用《Spring in Action》中的一句话：获取器注入是一种特殊的方法注人，它是把一个方法声明为返回某种类型的 bean，但实际要返回的 bean是在配置文件里面配置的，此方法可用在设计有些可插拔的功能上，解除程序依赖。我们看看具体的应用。

**( 1)首先我们创建一个父类。**

```
public class User {
    
    public void showMe () {
		system.out.println( "i am user");
    }
}
```

**( 2）创建调用方法。**

```
public abstract class GetBeanTest {
    
public void showMe () {
	this.getBean().showMe ( ) ;
}
public abstract User getBean ();//这里没有任何实现
```

**（3）配置文件**

```
<bean id="getBeanTest" class="test.lookup.app.GetBeanTest">
<lookup-method name="getBean" bean="teacher"/>
</bean>
<bean id="teacher" class="test.lookup.bean.Teacher"/>
```

**（4）创建测试**

```
public static void main (string [] args) {
Applicationcontext bf = new ClassPathXmlApplicationContext ("lookupTest.xml");
GetBeanTest test=(GetBeanTest)bf.getBean ("getBeanTest");
test.showMe () ;
```

在配置文件中，我们看到了源码解析中提到的lookup-method子元素，这个配置完成的功能是动态地将teacher所代表的bean作为getBean的返回值，运行测试方法我们会看到控制台上的输出:

i am Teacher

### replace-method

方法替换:可以在运行时用新的方法替换现有的方法。与之前的 look-up不同的是,replaced-method不但可以动态地替换返回实体 bean，而且还能动态地更改原有方法的逻辑。我们来看看使用示例。

**(1)在changeMe中完成某个业务逻辑。**

```
public class TestChangeMethod {
    
public void changeMe() {
	system.out.println (" changeMe" ) ;
}
    
}
```

**(2)在运营一段时间后需要改变原有的业务逻辑。注意：需要实现MethodReplacer接口，重写reimplement方法**

```
public class TestMethodReplacer implements MethodReplacer{
   @override
	public Object reimplement(Object obj，Method method，Object [] args)throws Throwable {
			System. out.println("我替换了原有的方法");
    }
}
```

**( 3 ）使替换后的类生效。**

```
<bean id="testChangeMethod" class="test.replacemethod.TestChangeMethod">
<replaced-method name="changeMe" replacer="replacer" />
</bean>

<bean id="replacer" class="test.replacemethod.TestMethodReplacer"/>
```

**（4）测试**

```
public static void main (string[] args) {
ApplicationContext bf = new ClassPathXmlApplicationContext ("replaceMethodTest.xml" );
TestChangeMethod test=(TestChangeMethod) bf.getBean("testChangeMethod");
test.changeMe () ;//输出：我替换了原有的方法
```

### Bean作用域

#### singleton和prototype

1.  在 Spring 里面，默认情况下，bean 是单实例对象 ，即多次调用

```
context.getBean("user", User.class);
```

所返回的对象实例是同一个

2.  如何设置单实例还是多实例 

（1）在 spring 配置文件 bean 标签里面有属性（scope）用于设置单实例还是多实例 

（2）scope 属性值，默认值为singleton，表示是单实例对象

第二个值 prototype，表示是多实例对象 

```
<bean id="user" class="com.wu.spring.User" scope= "prototype"></bean>
```



3.  （1）singleton 和 prototype 区别 第一 singleton 单实例，prototype 多实例 

（2）设置 scope 值是 singleton 时候，加载 spring 配置文件时候就会创建单实例对象

设置 scope 值是 prototype 时候，不是在加载 spring 配置文件时候创建 对象，而是在调用 getBean 方

法时候创建多实例对象 

4.  对于singleton，容器创建的对象的生命周期几乎和容器一样，只要容器不销毁或退出，对象就不会被回收

而对于prototype，容器所创建的每个对象的回收都由请求方控制，容器对该对象不持有任何控制权

#### request、session和global session

这三个scope类型是Spirng 2.0之后新增加的，它们不像之前的singleton和prototype那么“通用”， 因为它们只适用于Web应用程序，通常是与XmlWebApplicationContext共同使用。

-   **request**： request通常的配置形式如下： 

```
<bean id="requestProcessor" class="...RequestProcessor" scope="request"/>
```

Spring容器，即XmlWebApplicationContext会为每个HTTP请求创建一个全新的RequestProcessor对象供当前请求使用，当请求结束后，该对象实例的生命周期即告结束。当同时有10个HTTP 请求进来的时候，容器会分别针对这10个请求返回10个全新的RequestProcessor对象实例，且它们 之间互不干扰。从不是很严格的意义上说，request可以看作prototype的一种特例，除了场景更加具体 之外，语意上差不多。 

-   **session**

对于Web应用来说，放到session中的最普遍的信息就是用户的登录信息，对于这种放到session中 的信息，我们可使用如下形式指定其scope为session： 

```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
```

Spring容器会为每个独立的session创建属于它们自己的全新的UserPreferences对象实例。与 request相比，除了拥有session scope的bean的实例具有比request scope的bean可能更长的存活时间，其 他方面真是没什么差别

-   **global**

global session只有应用在基于portlet的Web应用程序中才有意义，它映射到portlet的global范围的 session。如果在普通的基于servlet的Web应用中使用了这个类型的scope，容器会将其作为普通的session 类型的scope对待 

#### 自定义scope

先不写，书讲得不够详细

### Bean生命周期

bean 生命周期 ：

（1）通过构造器创建 bean 实例（无参数构造） 

（2）为 bean 的属性设置值和对其他 bean 引用（调用 set 方法） 

（3）调用 bean 的初始化的方法（需要进行配置初始化的方法） 

（4）bean 可以使用了（对象获取到了） 

（5）当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法） 

演示：

```
public class Orders {
    //无参数构造
 public Orders() {
	 System.out.println("第一步 执行无参数构造创建 bean 实例");
 }
    
 private String oname;
    
 public void setOname(String oname) {
 	this.oname = oname;
	 System.out.println("第二步 调用 set 方法设置属性值");
 }
    
 //创建执行的初始化的方法
 public void initMethod() {
 	System.out.println("第三步 执行初始化的方法");
 }
 //创建执行的销毁的方法
 public void destroyMethod() {
	 System.out.println("第五步 执行销毁的方法");
 }
}
```

```
<bean id="orders" class="com.atguigu.spring5.bean.Orders" 
      init-method="initMethod" destroy-method="destroyMethod">
 <property name="oname" value="手机"></property>
</bean>
```

```
@Test
public void testBean3() {
 ClassPathXmlApplicationContext context =
 	new ClassPathXmlApplicationContext("bean4.xml");
 Orders orders = context.getBean("orders", Orders.class);
 System.out.println("第四步 获取创建 bean 实例对象");
 System.out.println(orders);
 //手动让 bean 实例销毁
 context.close();
 }
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ca11c93e6a64d87ae1306c2dab2095e~tplv-k3u1fbpfcp-zoom-1.image)



当添加了BeanPostProcessor后，有七步

BeanPostProcessor接口作用：

如果我们想在Spring容器中完成bean实例化、配置以及其他初始化方法前后要添加一些自己逻辑处理。我们需要定义一个或多个BeanPostProcessor接口实现类，然后注册到Spring IoC容器中。

```
public class MyBeanPost implements BeanPostProcessor {
 @Override
 public Object postProcessBeforeInitialization(Object bean, String beanName) 
throws BeansException {
 System.out.println("在初始化之前执行的方法");
 return bean;
 }
 @Override
 public Object postProcessAfterInitialization(Object bean, String beanName) 
throws BeansException {
 System.out.println("在初始化之后执行的方法");
 return bean;
 }
}
```

```
<!--配置后置处理器-->
<bean id="myBeanPost" class="com.atguigu.spring5.bean.MyBeanPost"></bean>
```

ApplicationContex会自动识别容器里的BeanPostProcessor实现类，而BeanFactory需要显式声明：

```
ConfigurableBeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource(...)); 
beanFactory.addBeanPostProcessor(new PasswordDecodePostProcessor());
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c8229a481fa4413ae0fae59230a6af1~tplv-k3u1fbpfcp-zoom-1.image)
### 自动装配

1、什么是自动装配 

根据指定装配规则（属性名称或者属性类型），Spring 自动将匹配的属性值进行注入 

2、演示自动装配过程 

```
<!--实现自动装配
 bean标签的autowire属性负责自动装配
 autowire 属性常用两个值：
 byName 根据属性名称注入 ，注入值 bean 的 id 值和类属性名称一样。若有多个一样的beanid，则无法自
				动注入
 byType 根据属性类型注入，若有多个一样类型的Bean，则无法自动注入
-->

<!-- byName根据属性名进行自动装配-->
<bean id="emp" class="com.atguigu.spring5.autowire.Emp" autowire="byName">
	 <!--相当于<property name="dept" ref="dept"></property>-->
</bean>
<bean id="dept" class="com.atguigu.spring5.autowire.Dept"></bean>

<!-- byType根据类型进行自动装配-->
<!-- 会找到类型为Dept的Bean进行注入-->
<bean id="emp" class="com.atguigu.spring5.autowire.Emp" autowire="byType">
 <!--<property name="dept" ref="dept"></property>-->
</bean>
<bean id="dept" class="com.atguigu.spring5.autowire.Dept"></bean>
```

### IOC 操作 Bean 管理(外部属性文件) 

看pdf笔记



### 使用parent属性和abstract实现模板化配置

当某个bean标签中设置了parent属性，则表示该bean继承了parent的属性值，即二者对于相同属性有相同的值或相同的注入对象

abstract属性的bean不需要声明class，表明该bean只是一个模板

```
<bean id="newsProviderTemplate" abstract="true"> 
 <property name="newPersistener"> 
 <ref bean="djNewsPersister"/> 
 </property> 
</bean> 

<bean id="superNewsProvider" parent="newsProviderTemplate" 
 class="..FXNewsProvider"> 
 <property name="newsListener"> 
 <ref bean="djNewsListener"/> 
 </property> 
</bean>

<bean id="subNewsProvider" parent="newsProviderTemplate" 
 class="..SpecificFXNewsProvider"> 
 <property name="newsListener"> 
 <ref bean="specificNewsListener"/> 
 </property> 
</bean> 
```



### 工厂方法与FactoryBean

#### 静态工厂方法

只看xml

```
 
<bean id="foo" class="...Foo"> 
 <property name="barInterface"> 
 <ref bean="bar"/> 
 </property> 
</bean> 

<bean id="bar" class="...StaticBarInterfaceFactory" factory-method="getInstance"/>
```

在客户端中，barInterface是第三方类提供的接口，为了解除客户端和第三方库实现类的过度耦合，使用了一个StaticBarInterfaceFactory类，将与第三方库接口实现类的直接耦合放在这个类里，使用这个类的getInstance返回第三方库接口的实现类给客户端

#### 非静态工厂方法

```
<bean id="foo" class="...Foo"> 
 <property name="barInterface"> 
 <ref bean="bar"/>
 </property> 
</bean> 

<bean id="barFactory" class="...NonStaticBarInterfaceFactory"/> 

<bean id="bar" factory-bean="barFactory" factory-method="getInstance"/> 
```

使用factory-bean属性来指定工厂方法所在的工厂类实例，而不是通过 class属性来指定工厂方法所在类的类型。指定工厂方法名则相同，都是通过factory-method属性进行的。

#### FactoryBean

注意：这是一个Spring提供的接口

要实现并使用自己的 FactoryBean其实很简单， org.springframework.beans.factory. FactoryBean只定义了三个方法，如以下代码所示：

```
 public interface FactoryBean { 
     Object getObject() throws Exception;
     Class getObjectType(); 
     boolean isSingleton(); 
 }
```

getObject()方法会返回该FactoryBean“生产”的对象实例，我们需要实现该方法以给出自己的对象实例化逻辑；getObjectType()方法仅返回getObject()方法所返回的对象的类型，如果预先 无法确定，则返回null；isSingleton()方法返回结果用于表明，工厂方法（getObject()）所“生产”的对象是否要以singleton形式存在于容器中。如果以singleton形式存在，则返回true，否则返回false； 

```
<bean id="nextDayDateDisplayer" class="...NextDayDateDisplayer"> 
 <property name="dateOfNextDay"> 
 <ref bean="nextDayDate"/> 
 </property> 
</bean> 
 
<bean id="nextDayDate" class="...NextDayDateFactoryBean"> 
</bean> 
```

咋看之下与普通的Bean没什么区别，但是注意，NextDayDateDisplayer类中的dateOfNextDay是DateTime类型而不是NextDayDateFactoryBean类型，**即Spring会自动识别FactoryBean的实现类，并且将该实现类中getObject()方法返回的对象注入依赖对象。**

**注：** 若要返回NextDayDateFactoryBean类型，则在getbean("beanName"); 的beanName前加上"&"：

getbean("&nextDayDate");

****

## 基于注解方式

Spring 针对 Bean 管理中创建对象提供注解\
（1）@Component  \
（2）@Service  \
（3）@Controller  \
（4）@Repository 

上面四个注解功能是一样的，都可以用来创建 bean 实例

### 半注解方式

步骤：

1.  引入相关jar包

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2369f399a7a345e0a2fab24297d14268~tplv-k3u1fbpfcp-zoom-1.image)

2.  开启组件扫描

```
<context:component-scan base-package="com.atguigu"></context:component-scan>
```

可以加一些配置

```
<!-- 配置扫描注解,不扫描@Controller注解 -->
<context:component-scan base-package="com.atguigu">
    <context:exclude-filter type="annotation"
        expression="org.springframework.stereotype.Controller" />
</context:component-scan>
```

3.  创建类，在类上面添加创建对象注解 

```
//在注解里面 value 属性值可以省略不写，
//默认值是类名称，首字母小写
//UserService -- userServic

@Service(value = "userService")//相当于<bean id="userService" class=".."/>
public class UserService {
    
 //定义 dao 类型属性
 //不需要添加 set 方法
 //添加注入属性注解@Autowired
 @Autowired 
 private UserDao userDao;
 public void add() {
	 System.out.println("service add.......");
 	userDao.add();
   }
}
```

当UserDao有多个实现类时，需要用@Qualifier(value="xxx")指定对象

```
@Autowired //根据类型进行注入
@Qualifier(value = "userDaoImpl1") //根据名称进行注入
private UserDao userDao;
```

### 全注解方式

步骤：

1.  创建配置类，替代 xml 配置文件

```
@Configuration //作为配置类，替代 xml 配置文件
@ComponentScan(basePackages = {"com.atguigu"})
public class SpringConfig {
}
```

2.  编写测试类

```
@Test
public void testService2() {
 //加载配置类
 ApplicationContext context = 
     new AnnotationConfigApplicationContext(SpringConfig.class);//只有这句不同
 UserService userService = context.getBean("userService", UserService.class);
 System.out.println(userService);
 userService.add();
}
```

若有收获，就点个赞吧

