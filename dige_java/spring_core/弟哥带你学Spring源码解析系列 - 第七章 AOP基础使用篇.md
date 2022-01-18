## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

这里的使用是基于AspectJ实现AOP操作，与原理关系不大

**AspectJ表达式：**

（1）切入点表达式作用：知道对哪个类里面的哪个方法进行增强\
（2）语法结构： execution([权限修饰符] [返回类型] [类全路径] 方法名称 )

举例 1：对 com.atguigu.dao.BookDao 类里面的 add 进行增强

execution(* com.atguigu.dao.BookDao.add(..)) 

举例 2：对 com.atguigu.dao 包里面所有类，类里面所有方法进行增强 

execution(* com.atguigu.dao.*.* (..)) 

****

**首先引入依赖：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9bab3656243341378b17aeb4d82decc6~tplv-k3u1fbpfcp-zoom-1.image)

# AOP操作（XML配置文件）

## 1、创建两个类，增强类和被增强类，创建方法 

```
public class User {
 public void add() {
 System.out.println("add.......");
 }
}
```

```
public class UserProxy {
 public void before() {//前置通知
 System.out.println("before......");
 }
}
```




## 2、在 spring 配置文件中创建两个类对象 




```
<!--创建对象-->
<bean id="book" class="com.atguigu.spring5.aopxml.User"></bean>
<bean id="bookProxy" class="com.atguigu.spring5.aopxml.UserProxy"></bean>
```




## 3、在 spring 配置文件中配置切入点 

```
<!--配置 aop 增强-->
<aop:config>
 <!--切入点-->
 <aop:pointcut id="p" expression="execution(* 
com.atguigu.spring5.aopxml.Book.buy(..))"/>
 <!--配置切面-->
 <aop:aspect ref="bookProxy">
 <!--增强作用在具体的方法上-->
 <aop:before method="before" pointcut-ref="p"/>
 </aop:aspect>
</aop:config>
```




# AOP操作(注解)：

## 1、创建类，在类里面定义方法 

```
@Component
public class User {
 public void add() {
 System.out.println("add.......");
 }
}
```

## 2、创建增强类（编写增强逻辑） 

在增强类里面，创建方法，让不同方法代表不同通知类型 

```
@Aspect
@Component
public class UserProxy {
 public void before() {//前置通知
 System.out.println("before......");
 }
}
```

## 3、进行通知的配置 

### （1）在 spring 配置文件中，开启注解扫描

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" 
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 xmlns:context="http://www.springframework.org/schema/context" 
 xmlns:aop="http://www.springframework.org/schema/aop" 
 xsi:schemaLocation="http://www.springframework.org/schema/beans 
http://www.springframework.org/schema/beans/spring-beans.xsd 
 http://www.springframework.org/schema/context 
http://www.springframework.org/schema/context/spring-context.xsd 
 http://www.springframework.org/schema/aop 
http://www.springframework.org/schema/aop/spring-aop.xsd">
  
 <!-- 开启注解扫描 -->
 <context:component-scan basepackage="com.atguigu.spring5.aopanno"></context:component-scan>
```

### （2）使用注解创建 User 和 UserProxy 对象 

即前两步中的@Component注解

### （3）在增强类上面添加注解 @Aspect 

即第二步中的@Aspect注解

## 4、配置不同类型的通知 

**在增强类的里面，在作为通知方法上面添加通知类型注解，使用切入点表达式配置**

```
@Component
@Aspect //生成代理对象
public class UserProxy {
 //前置通知
 //@Before 注解表示作为前置通知
 @Before(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
 public void before() {
 System.out.println("before.........");
 }
    
 //后置通知（返回通知）
 @AfterReturning(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
 public void afterReturning() {
 System.out.println("afterReturning.........");
 }
    
 //最终通知
 @After(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
 public void after() {
 System.out.println("after.........");
 }
    
 //异常通知
 @AfterThrowing(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
 public void afterThrowing() {
 System.out.println("afterThrowing.........");
 }
    
 //环绕通知
 @Around(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
 public void around(ProceedingJoinPoint proceedingJoinPoint) throws 
Throwable {
 System.out.println("环绕之前.........");
 //被增强的方法执行
 proceedingJoinPoint.proceed();
 System.out.println("环绕之后.........");
 }
}
```

## 5、相同的切入点抽取 

```
//相同切入点抽取
@Pointcut(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
public void pointdemo() {
}

//前置通知
//@Before 注解表示作为前置通知
@Before(value = "pointdemo()")
public void before() {
 System.out.println("before.........");
}
```

## 6、有多个增强类对同一个方法进行增强，设置增强类优先级 

在增强类上面添加注解 @Order(数字类型值)，数字类型值越小优先级越高 

```
@Component 
@Aspect 
@Order(1) 
public class PersonProxy{...}  
```

# 完全注解开发

创建配置类，不需要创建 xml 配置文件 

```
@Configuration
@ComponentScan(basePackages = {"com.atguigu"})
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class ConfigAop {
}
```

和IOC的完全注解开发一样，只需要在启动类加个注解依赖到配置类就行

