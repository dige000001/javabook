## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 基本概念

### JointPoint

在系统运行之前，AOP的功能模块都需要织入到OOP的功能模块中。所以，要进行这种织入过程， 我们需要知道在系统的哪些执行点上进行织入操作，这些将要在其之上进行织入操作的系统执行点就称之为Joinpoint。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b3a1cc845ca410cb0774b9053804e09~tplv-k3u1fbpfcp-zoom-1.image)

**方法调用（Method Call）** 。当某个方法被调用的时候所处的程序执行点，图7-6中的后面三个“圆 圈”所标记的时点都属于这种类型。

**方法调用执行（Method Call execution）** 。称之为方法执行或许更简洁，该Joinpoint类型代表的是**某个方法内部执行开始时点**

方法调用（method call）是在调用对象上的执行点，而方法执行（method execution）则是在被调 用到的方法逻辑执行的时点。对于同一对象，方法调用要先于方法执行。 

注：Spring仅支持**方法调用执行（Method Call execution）** ，不过可以使用AspectJ支持其他JointPoint。

### PointCut

通俗地说：PointCut用来说明要在哪些JointPoint中织入横切逻辑 

当前的AOP产品所使用的Pointcut表达形式通常可以简单划分为以下几种：

1.  直接指定Joinpoint所在方法名称。 这种方式通常只限于Joinpoint较少且较为简单的情况。 
1.  正则表达式

<!---->

3.  使用特定的Pointcut表述语言

### Advice

Advice是单一横切关注点逻辑的载体，**它代表将会织入到Joinpoint的横切逻辑**。 

按照Advice在Joinpoint位置执行时机的差异或者完成功能的不同，Advice可以分成多种具体形式：

1.  **Before Advice**

Before Advice是在Joinpoint指定位置之前执行的Advice类型。通常，它不会中断程序执行流程， 但如果必要，可以通过在Before Advice中抛出异常的方式来中断当前程序流程。如果当前Before Advice 将被织入到方法执行类型的Joinpoint，那么这个Before Advice就会先于方法执行而执行。 通常，可以使用Before Advice做一些系统的初始化工作，比如设置系统初始值，获取必要系统资 源等。当然，并非就限于这些情况。如果要用Before Advice来封装安全检查的逻辑，也不是不可以的， 但通常情况下，我们会使用另一种形式的Advice 

2.  **After Advice**

顾名思义，After Advice就是在相应连接点之后执行的Advice类型，但该类型的Advice还可以细分 为以下三种：

**After returning Advice**。只有当前Joinpoint处执行流程正常完成后，After returning Advice才会 执行。比如方法执行正常返回而没有抛出异常。  

**After throwing Advice**。又称Throws Advice，只有在当前Joinpoint执行过程中抛出异常的情况 下，才会执行。比如某个方法执行类型的Joinpoint抛出某异常而没有正常返回。  

**After Advice**。或许叫After (Finally) Advice更为确切，该类型Advice不管Joinpoint处执行流程 是正常终了还是抛出异常都会执行，就好像Java中的finally块一样。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64f939ea38244c2198009d4d0b627990~tplv-k3u1fbpfcp-zoom-1.image)

3.  **Around Advice**

Around Advice对附加其上的Joinpoint进行“包裹”，可以在Joinpoint之前和之后都指定相应的逻辑， 甚至于中断或者忽略Joinpoint处原来程序流程的执行。 




4.  **Introduction**

与之前的几种Advice类型不同，Introduction不是根据横切逻辑在Joinpoint处的执行时机来区分的，而是根据它可以完成的功能而区别于其他Advice类型。 Introduction可以为原有的对象添加新的特性或者行为，这就好像你是一个普通公民，当让你穿军 装，带军帽，添加了军人类型的Introduction之后，你就拥有军人的特性或者行为 。

### Aspect 

Aspect是对系统中的横切关注点逻辑进行模块化封装的AOP概念实体。通常情况下，Aspect可以包 含多个Pointcut以及相关Advice定义。比如，以AspectJ形式定义的Aspect如下所示：

```
aspect AjStyleAspect 
{ 
 // pointcut定义
 pointcut query():call(public * get*(..)); 
 pointcut update(): execution(public void update*(..)); 
 ... 
 
 // advice定义
 before(): query() 
 { 
 ... 
 } 
 after() returnint: update() 
 { 
 ... 
 } 
 ... 
} 
```

Spring AOP最初没有“完全”确切的实体对应真正的Aspect的概念。在2.0发布后，因为集成了 AspectJ，所以可以通过使用@AspectJ的注解并结合普通的POJO来声明Aspect。

### 织入和织入器

AspectJ有专门的编译器来完成织入操作，即ajc，所以ajc就是AspectJ完成织入的织入器； JBoss AOP采用自定义的类加载器来完成最终织入，那么这个自定义的类加载器就是它的织入器； Spring AOP使用一组类来完成最终的织入操作，ProxyFactory类则是Spring AOP中最通用的织入器。总之， Java平台各AOP实现的织入器形式不一而足，唯一相同的就是它们的职责，即完成横切关注点逻辑到 系统的最终织入。 



# 动态代理

JDK提供了java.lang.reflect.InvocationHandler接口和 java.lang.reflect.Proxy类，这两个类相互配合，入口是Proxy。

Proxy有个静态方法：getProxyClass(ClassLoader, interfaces)，只要你给它传入类加载器和一组接口，它就给你返回代理Class对象。

用通俗的话说，getProxyClass()这个方法，会从你传入的接口Class中，“拷贝”类结构信息到一个新的Class对象中，但新的Class对象带有构造器，是可以创建对象的。打个比方，一个大内太监（接口Class），空有一身武艺（类信息），但是无法传给后人。现在江湖上有个妙手神医（Proxy类），发明了克隆大法（getProxyClass），不仅能克隆太监的一身武艺，还保留了小DD（构造器）

```
public class ProxyTest {
	public static void main(String[] args) throws Throwable {
		CalculatorImpl target = new CalculatorImpl();
                //传入目标对象
                //目的：1.根据它实现的接口生成代理对象 2.代理对象调用目标对象方法
		Calculator calculatorProxy = (Calculator) getProxy(target);
		calculatorProxy.add(1, 2);
		calculatorProxy.subtract(2, 1);
	}

	private static Object getProxy(final Object target) throws Exception {
		//参数1：随便找个类加载器给它， 参数2：目标对象实现的接口，让代理对象实现相同接口
		Class proxyClazz = Proxy.getProxyClass(
            			   						 target.getClass().getClassLoader(), 													 target.getClass().getInterfaces()
                                              );
        //代理Class对象有一个构造器，参数是InvocationHandler，这里返回这个构造器
		Constructor constructor = proxyClazz.getConstructor(InvocationHandler.class);
		Object proxy = constructor.newInstance(new InvocationHandler() {
			@Override
		public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            
				System.out.println(method.getName() + "方法开始执行...");
				Object result = method.invoke(target, args);
				System.out.println(result);
				System.out.println(method.getName() + "方法执行结束...");
				return result;
			}
		});
		return proxy;
	}
}
```

在上面的代码中，代理Class对象返回的代理对象中有一个InvocationHandler，每次调用代理对象的方法，最终都会调用InvocationHandler的invoke()方法：




不过实际编程中，一般不用getProxyClass()，而是使用Proxy类的另一个静态方法：Proxy.newProxyInstance()，直接返回代理实例，连中间得到代理Class对象的过程都帮你隐藏：

举个例子：

假设有一个subject接口和实现类：

```
public interface Subject   
{   
  public void doSomething();   
}
```

```
public class RealSubject implements Subject   
{   
  public void doSomething()   
  {   
    System.out.println( "call doSomething()" );   
  }   
}
```

我们想在doSomething()前后加一些增强逻辑，使用InvocationHandler和Proxy类进行动态代理：

我们需要实现InvocationHandler接口

```
import java.lang.reflect.InvocationHandler;  
import java.lang.reflect.Method;  
import java.lang.reflect.Proxy;  

public class ProxyHandler implements InvocationHandler
{
    private Object tar;
      public ProxyHandler(Object target) {
        this.tar = target;
    }
    
 	public Object getProxy() {
   	return Proxy.newProxyInstance(tar.getClass().getClassLoader(),
                                      tar.getClass().getInterfaces(),
                                      this);
    }
    
   

    public Object invoke(Object proxy , Method method , Object[] args)throws Throwable
    {
        Object result = null;
        
        //这里就可以进行所谓的AOP编程了
        //you can do something before the method called
        
        result = method.invoke(tar,args);
        
        //you can do something after the method called
        return result;
    }
}
```

```
public class TestProxy
{
    public static void main(String args[])
    {
           ProxyHandler proxy = new ProxyHandler(new RealSubject());
           //传入实现类
           Subject sub = (Subject) proxy.getProxy();
           sub.doSomething();
    }
}
```

在这个过程中，

动态代理的作用是什么：

1.  Proxy类的代码量被固定下来，不会因为业务的逐渐庞大而庞大；
1.  可以实现AOP编程，实际上静态代理也可以实现，总的来说，AOP可以算作是代理模式的一个典型应用；

<!---->

3.  解耦，通过参数就可以判断真实类，不需要事先实例化，更加灵活多变。




# CGLIB

动态代理适用于有接口的实现类，但对于没有接口的类，动态代理无法完成方法增强，此时需要CGLIB字节码生成完成这个功能。

CGLIB会给被代理类生成一个子类，把这个子类作为代理类，故对于被final修饰的类，CGLIB无法代理。

**CGLIB原理**：动态生成一个要代理类的子类，子类重写要代理的类的所有不是final的方法。在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。它比使用java反射的JDK动态代理要快。

**CGLIB底层**：使用字节码处理框架ASM，来转换字节码并生成新的类。不鼓励直接使用ASM，因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉。

[\
\
\
\
](https://blog.csdn.net/zghwaicsdn/article/details/50957474/)

例子：

我们有一个TargetObject类

```
public class TargetObject {
	public String method1(String paramName) {
		return paramName;
	}
 
	public int method2(int count) {
		return count;
	}
 
	public int method3(int count) {
		return count;
	}
 
	@Override
	public String toString() {
		return "TargetObject []"+ getClass();
	}
}
```

然后实现拦截器MethodInterceptor接口

```
import java.lang.reflect.Method;
 
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
public class TargetInterceptor implements MethodInterceptor{
 
	/**
	 * 重写方法拦截在方法前和方法后加入业务
	 * Object obj为目标对象
	 * Method method为目标方法
	 * Object[] params 为参数，
	 * MethodProxy proxy CGlib方法代理对象
	 */
	@Override
	public Object intercept(Object obj, Method method, Object[] params,
			MethodProxy proxy) throws Throwable {
		System.out.println("调用"+method.getName()+"前");
        
        // 注意这里是调用invokeSuper而不是invoke，否则死循环;
        // methodProxy.invokeSuper执行的是原始类的方法;
        // method.invoke执行的是代理子类的方法;
		Object result = proxy.invokeSuper(obj, params);
		System.out.println(" 调用"+method.getName()+"后"+result);
		return result;
	}
}
```

测试：

```
import net.sf.cglib.proxy.Callback;
import net.sf.cglib.proxy.CallbackFilter;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.NoOp;
 
public class TestCglib {
	public static void main(String args[]) {
        // 创建Enhancer对象，类似于JDK动态代理的Proxy类
		Enhancer enhancer =new Enhancer();
        // 设置目标类的字节码文件
		enhancer.setSuperclass(TargetObject.class);
        // 设置回调函数
		enhancer.setCallback(new TargetInterceptor());
        // create方法正式创建代理类
		TargetObject targetObject2=(TargetObject)enhancer.create();
		System.out.println(targetObject2);
		System.out.println(targetObject2.method1("mmm1"));
		System.out.println(targetObject2.method2(100));
		System.out.println(targetObject2.method3(200));
	}
}
```

CGLIB相比动态代理有一个优点：可以设置对不同方法执行不同的回调逻辑，或者根本不执行回调。

首先我们要实现一个过滤器：根据返回值的不同调用Callback数组的不同回调方法，即不同的增强逻辑

```
import java.lang.reflect.Method;
import net.sf.cglib.proxy.CallbackFilter;
public class TargetMethodCallbackFilter implements CallbackFilter {
 
	/**
	 * 过滤方法
	 * 返回的值为数字，代表了Callback数组中的索引位置，要到用的Callback
	 */
	@Override
	public int accept(Method method) {
		if(method.getName().equals("method1")){
			System.out.println("filter method1 ==0");
			return 0;
		}
		if(method.getName().equals("method2")){
			System.out.println("filter method2 ==1");
			return 1;
		}
		if(method.getName().equals("method3")){
			System.out.println("filter method3 ==2");
			return 2;
		}
		return 0;
	}
}
```

测试：

```
import net.sf.cglib.proxy.Callback;
import net.sf.cglib.proxy.CallbackFilter;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.NoOp;
 
public class TestCglib {
	public static void main(String args[]) {
		Enhancer enhancer =new Enhancer();
		enhancer.setSuperclass(TargetObject.class);
		CallbackFilter callbackFilter = new TargetMethodCallbackFilter();
		/**
		 * (1)callback1：方法拦截器
		   (2)NoOp.INSTANCE：这个NoOp表示no operator，即什么操作也不做，代理类直接调用被代理的							 方法不进行拦截。
		   (3)FixedValue：表示锁定方法返回值，无论被代理类的方法返回什么值，回调方法都返回固定                             值。
		 */
		Callback noopCb=NoOp.INSTANCE;
		Callback callback1=new TargetInterceptor();
		Callback fixedValue=new TargetResultFixed(999);
		Callback[] cbarray=new Callback[]{callback1,noopCb,fixedValue};
        
       //设定回调方法数组
		enhancer.setCallbacks(cbarray);
        //设定过滤器
		enhancer.setCallbackFilter(callbackFilter);
		TargetObject targetObject2=(TargetObject)enhancer.create();
		System.out.println(targetObject2);
		System.out.println(targetObject2.method1("mmm1"));
		System.out.println(targetObject2.method2(100));
		System.out.println(targetObject2.method3(100));
		System.out.println(targetObject2.method3(200));
	}
}
```

## 原理

CglibAopProxy的入口应该是在getProxy，也就是说在CglibAopProxy类的getProxy方法中实现了Enhancer的创建及接口封装。

```
@Override
public Object getProxy(@Nullable ClassLoader classLoader) {
    if (logger.isDebugEnabled()) {
        logger.debug("Creating CGLIB proxy: target source is " + this.advised.getTargetSource());
    }

    try {
        Class<?> rootClass = this.advised.getTargetClass();
        Assert.state(rootClass != null, "Target class must be available for creating a CGLIB proxy");

        Class<?> proxySuperClass = rootClass;
        if (ClassUtils.isCglibProxyClass(rootClass)) {
            proxySuperClass = rootClass.getSuperclass();
            Class<?>[] additionalInterfaces = rootClass.getInterfaces();
            for (Class<?> additionalInterface : additionalInterfaces) {
                this.advised.addInterface(additionalInterface);
            }
        }

        // Validate the class, writing log messages as necessary.
        // 验证Class
        validateClassIfNecessary(proxySuperClass, classLoader);

        // Configure CGLIB Enhancer...
        // 创建及配置Enhancer
        Enhancer enhancer = createEnhancer();
        if (classLoader != null) {
            enhancer.setClassLoader(classLoader);
            if (classLoader instanceof SmartClassLoader &&
                    ((SmartClassLoader) classLoader).isClassReloadable(proxySuperClass)) {
                enhancer.setUseCache(false);
            }
        }
        enhancer.setSuperclass(proxySuperClass);
        enhancer.setInterfaces(AopProxyUtils.completeProxiedInterfaces(this.advised));
        enhancer.setNamingPolicy(SpringNamingPolicy.INSTANCE);
        enhancer.setStrategy(new ClassLoaderAwareUndeclaredThrowableStrategy(classLoader));

        // 设置拦截器
        Callback[] callbacks = getCallbacks(rootClass);
        Class<?>[] types = new Class<?>[callbacks.length];
        for (int x = 0; x < types.length; x++) {
            types[x] = callbacks[x].getClass();
        }
        // fixedInterceptorMap only populated at this point, after getCallbacks call above
        enhancer.setCallbackFilter(new ProxyCallbackFilter(
                this.advised.getConfigurationOnlyCopy(), this.fixedInterceptorMap, this.fixedInterceptorOffset));
        enhancer.setCallbackTypes(types);

        // Generate the proxy class and create a proxy instance.
        // 生成代理类以及创建代理
        return createProxyClassAndInstance(enhancer, callbacks);
    }
    catch (CodeGenerationException | IllegalArgumentException ex) {
        throw new AopConfigException("Could not generate CGLIB subclass of " + this.advised.getTargetClass() +
                ": Common causes of this problem include using a final class or a non-visible class",
                ex);
    }
    catch (Throwable ex) {
        // TargetSource.getTarget() failed
        throw new AopConfigException("Unexpected AOP exception", ex);
    }
}

protected Object createProxyClassAndInstance(Enhancer enhancer, Callback[] callbacks) {
    enhancer.setInterceptDuringConstruction(false);
    enhancer.setCallbacks(callbacks);
    return (this.constructorArgs != null && this.constructorArgTypes != null ?
            enhancer.create(this.constructorArgTypes, this.constructorArgs) :
            enhancer.create());
}
```

以上函数完整地阐述了一个创建Spring中的Enhancer的过程，读者可以参考Enhancer的文档查看每个步骤的含义，这里最重要的是通过getCallbacks方法设置拦截器链。

```
private Callback[] getCallbacks(Class<?> rootClass) throws Exception {
    // Parameters used for optimization choices...
    // 对于expose-proxy属性的处理
    boolean exposeProxy = this.advised.isExposeProxy();
    boolean isFrozen = this.advised.isFrozen();
    boolean isStatic = this.advised.getTargetSource().isStatic();

    // Choose an "aop" interceptor (used for AOP calls).
    // 将拦截器封装在DynamicAdvisedInterceptor中
    Callback aopInterceptor = new DynamicAdvisedInterceptor(this.advised);

    // Choose a "straight to target" interceptor. (used for calls that are
    // unadvised but can return this). May be required to expose the proxy.
    Callback targetInterceptor;
    if (exposeProxy) {
        targetInterceptor = (isStatic ?
                new StaticUnadvisedExposedInterceptor(this.advised.getTargetSource().getTarget()) :
                new DynamicUnadvisedExposedInterceptor(this.advised.getTargetSource()));
    }
    else {
        targetInterceptor = (isStatic ?
                new StaticUnadvisedInterceptor(this.advised.getTargetSource().getTarget()) :
                new DynamicUnadvisedInterceptor(this.advised.getTargetSource()));
    }

    // Choose a "direct to target" dispatcher (used for
    // unadvised calls to static targets that cannot return this).
    Callback targetDispatcher = (isStatic ?
            new StaticDispatcher(this.advised.getTargetSource().getTarget()) : new SerializableNoOp());

    Callback[] mainCallbacks = new Callback[] {
            // 将拦截器链加入Callback中
            aopInterceptor,  // for normal advice
            targetInterceptor,  // invoke target without considering advice, if optimized
            new SerializableNoOp(),  // no override for methods mapped to this
            targetDispatcher, this.advisedDispatcher,
            new EqualsInterceptor(this.advised),
            new HashCodeInterceptor(this.advised)
    };

    Callback[] callbacks;

    // If the target is a static one and the advice chain is frozen,
    // then we can make some optimizations by sending the AOP calls
    // direct to the target using the fixed chain for that method.
    if (isStatic && isFrozen) {
        Method[] methods = rootClass.getMethods();
        Callback[] fixedCallbacks = new Callback[methods.length];
        this.fixedInterceptorMap = new HashMap<>(methods.length);

        // TODO: small memory optimization here (can skip creation for methods with no advice)
        for (int x = 0; x < methods.length; x++) {
            List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(methods[x], rootClass);
            fixedCallbacks[x] = new FixedChainStaticTargetInterceptor(
                    chain, this.advised.getTargetSource().getTarget(), this.advised.getTargetClass());
            this.fixedInterceptorMap.put(methods[x].toString(), x);
        }

        // Now copy both the callbacks from mainCallbacks
        // and fixedCallbacks into the callbacks array.
        callbacks = new Callback[mainCallbacks.length + fixedCallbacks.length];
        System.arraycopy(mainCallbacks, 0, callbacks, 0, mainCallbacks.length);
        System.arraycopy(fixedCallbacks, 0, callbacks, mainCallbacks.length, fixedCallbacks.length);
        this.fixedInterceptorOffset = mainCallbacks.length;
    }
    else {
        callbacks = mainCallbacks;
    }
    return callbacks;
}
```

在getCallback中Spring考虑了很多情况，但是对于我们来说，只需要理解最常用的就可以了，比如将advised属性封装在DynamicAdvisedlnterceptor并加人在callbacks中，这么做的目的是什么呢，如何调用呢？在前面的示例中，我们了解到CGLIB中对于方法的拦截是通过将自定义的拦截器（实现Methodlnterceptor接口）加人Callback中并在调用代理时直接激活拦截器中的intercept方法来实现的，那么在getCallback中正是实现了这样一个目的，DynamicAdvisedlnterceptor继承自Methodlnterceptor，加人Callback中后，在再次调用代理时会直接调用DynamicAdvisedlnterceptor中的intercept方法，由此推断，对于CGLIB方式实现的代理，其核心逻辑必然在 DynamicAdvisedlnterceptor 中的 intercept 中。

```
@Override
@Nullable
public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
    Object oldProxy = null;
    boolean setProxyContext = false;
    Object target = null;
    TargetSource targetSource = this.advised.getTargetSource();
    try {
        if (this.advised.exposeProxy) {
            // Make invocation available if necessary.
            oldProxy = AopContext.setCurrentProxy(proxy);
            setProxyContext = true;
        }
        // Get as late as possible to minimize the time we "own" the target, in case it comes from a pool...
        target = targetSource.getTarget();
        Class<?> targetClass = (target != null ? target.getClass() : null);
        // 获取拦截器链
        List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
        Object retVal;
        // Check whether we only have one InvokerInterceptor: that is,
        // no real advice, but just reflective invocation of the target.
        if (chain.isEmpty() && Modifier.isPublic(method.getModifiers())) {
            // We can skip creating a MethodInvocation: just invoke the target directly.
            // Note that the final invoker must be an InvokerInterceptor, so we know
            // it does nothing but a reflective operation on the target, and no hot
            // swapping or fancy proxying.
            Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
            // 如果拦截器链为空则直接激活原方法
            retVal = methodProxy.invoke(target, argsToUse);
        }
        else {
            // We need to create a method invocation...
            // 进入链
            retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
        }
        retVal = processReturnType(proxy, target, method, retVal);
        return retVal;
    }
    finally {
        if (target != null && !targetSource.isStatic()) {
            targetSource.releaseTarget(target);
        }
        if (setProxyContext) {
            // Restore old proxy.
            AopContext.setCurrentProxy(oldProxy);
        }
    }
}
```

上述的实现与JDK方式实现代理中的invoke方法大同小异，都是首先构造链，然后封装此链进行串联调用，稍有些区別就是在JDK中直接构造ReflectiveMethodlnvocation，而在cglib 中使用 CglibMethodlnvocation。CglibMethodlnvocation 继承自 ReflectiveMethodlnvocation，但是proceed方法并没有重写。

# SpringAOP

## Spring AOP 中的PointCut

Spring中以接口定义org.springframework.aop.Pointcut作为其AOP框架中所有Pointcut的最 顶层抽象，该接口定义了两个方法用来帮助捕捉系统中的相应Joinpoint，并提供了一个TruePointcut 类型实例。如果Pointcut类型为TruePointcut，默认会对系统中的所有对象，以及对象上所有被支持 的Joinpoint进行匹配。org.springframework.aop.Pointcut接口定义如以下代码所示： 

```
public interface Pointcut { 
 ClassFilter getClassFilter(); 
 MethodMatcher getMethodMatcher(); 
 Pointcut TRUE = TruePointcut.INSTANCE; 
} 
```

### ClassFilter和MethodMatcher

ClassFilter和MethodMatcher分别用于匹配将被执行织入操作的对象以及相应的方法。之所以 将类型匹配和方法匹配分开定义，是因为可以重用不同级别的匹配定义，并且可以在不同的级别或者 相同的级别上进行组合操作，或者强制让某个子类只覆写（Override）相应的方法定义等。

#### ClassFilter

ClassFilter接口的作用是对Joinpoint所处的对象进行Class级别的类型匹配，其定义如下： 

```
public interface ClassFilter { 
 boolean matches(Class clazz); 
 ClassFilter TRUE = TrueClassFilter.INSTANCE; 
} 
```

当织入的目标对象的Class类型与Pointcut所规定的类型相符时，matches方法将会返回true，否 则，返回false，即意味着不会对这个类型的目标对象进行织入操作。比如，如果我们仅希望对系统 中Foo类型的类执行织入，则可以如下这样定义ClassFilter： 

```
public class FooClassFilter{ 
 public boolean matches(Class clazz){ 
 return Foo.class.equals(clazz); 
 } 
} 
```

当然，如果类型对我们所捕捉的Joinpoint无所谓，那么Pointcut中使用的ClassFilter可以直接使 用“ClassFilter TRUE = TrueClassFilter.INSTANCE;”。当Pointcut中返回的ClassFilter类型为该类型实例时，Pointcut的匹配将会针对系统中所有的目标类以及它们的实例进行。 

#### MethodMatcher

相对于ClassFilter的简单定义，MethodMatcher则要复杂得多。 

```
public interface MethodMatcher { 
 boolean matches(Method method, Class targetClass); 
 boolean isRuntime(); 
 boolean matches(Method method, Class targetClass, Object[] args); 
 MethodMatcher TRUE = TrueMethodMatcher.INSTANCE; 
} 
```

MethodMatcher通过重载（Overload），定义了两个matches方法，而这两个方法的分界线就是 isRuntime()方法。在对对象具体方法进行拦截的时候，可以忽略每次方法执行的时候调用者传入的参数，也可以每次都检查这些方法调用参数，以强化拦截条件。

在MethodMatcher类型的基础上，Pointcut可以分为两类，即StaticMethodMatcherPointcut和 DynamicMethodMatcherPointcut。因为StaticMethodMatcherPointcut具有明显的性能优势，所 以，Spring为其提供了更多支持。图9-1给出了Spring AOP中各Pointcut类型之间的一个局部“族谱” 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce29688f4646413dbecb0faec0d56499~tplv-k3u1fbpfcp-zoom-1.image)



假设对以下方法进行拦截： 

public boolean login(String username,Sring password); 

如果只想在login方法之前插入计数功能，那么login方法的参数对于Joinpoint捕捉就是可以忽略的。而如果想在用户登录的时候对某个用户做单独处理，如不让其登录或者给予特殊权限，那么这个方法的参数就是在匹配Joinpoint的时候必须要考虑的。 

(1) 在前一种情况下，isRuntime返回false，表示不会考虑具体Joinpoint的方法参数，这种类型的MethodMatcher称之为StaticMethodMatcher。因为不用每次都检查参数，那么对于同样类型的方 法匹配结果，就可以在框架内部缓存以提高性能。isRuntime方法返回false表 明 当 前的 MethodMatcher为StaticMethodMatcher的时候，只有boolean matches(Method method, Class targetClass);方法将被执行，它的匹配结果将会成为其所属的Pointcut主要依据。 

(2) 当isRuntime方法返回true时，表明该MethodMatcher将会每次都对方法调用的参数进行匹配检查，这种类型的MethodMatcher称之为DynamicMethodMatcher。因为每次都要对方法参数进行检查，无法对匹配的结果进行缓存，所以，匹配效率相对于StaticMethodMatcher来说要差。而且大 部分情况下，StaticMethodMatcher已经可以满足需要，最好避免使用DynamicMethodMatcher类型。 如果一个MethodMatcher为DynamicMethodMatcher（isRuntime()返回true），并且当方法boolean matches(Method method, Class targetClass);也返回true的时候，三个参数的matches方法将 被执行，以进一步检查匹配条件。如果方法boolean matches(Method method,Class targetClass); 返回false，那么不管这个MethodMatcher是StaticMethodMatcher还是DynamicMethodMatcher， 该结果已经是最终的匹配结果——你可以猜得到，三个参数的matches方法那铁定是执行不了了。 

### 常见PointCut

《Spring揭秘》P146 （Real P160）

## Advice

Spring AOP加入了开源组织AOP Alliance（http://aopalliance.sourceforge.net/）①，目的在于标准化 AOP的使用，促进各个AOP实现产品之间的可交互性。鉴于此，Spring中的Advice实现全部遵循AOP Alliance规定的接口。图9-4中就是Spring中各种Advice类型实现与AOP Alliance中标准接口之间的关系

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b163310b60ce462689e5ad5d32a06b3a~tplv-k3u1fbpfcp-zoom-1.image)

Advice实现了将被织入到Pointcut规定的Joinpoint处的横切逻辑。在Spring中，Advice按照其自身 实例（instance）能否在目标对象类的所有实例中共享这一标准，可以划分为两大类，即per-class类型 的Advice和per-instance类型的Advice。 

### Per-Class

per-class类型的Advice是指，该类型的Advice的实例可以在目标对象类的所有实例之间共享。这种 类型的Advice通常只是提供方法拦截的功能，不会为目标对象类保存任何状态或者添加新的特性。 图9.4都是Per-Class类型的Advice

#### Before Advice

Before Advice所实现的横切逻辑将在相应的Joinpoint之前执行，在Before Advice执行完成之后， 程序执行流程将从Joinpoint处继续执行，所以Before Advice通常不会打断程序的执行流程。但是如果 必要，也可以通过抛出相应异常的形式中断程序流程。 要 在 Spring 中实现 Before Advice ，我们通常只需要实现 org.springframework.aop. MethodBeforeAdvice接口即可，该接口定义如下： 

```
public interface MethodBeforeAdvice extends BeforeAdvice { 
 void before(Method method, Object[] args, Object target) throws Throwable; 
} 
```

例子：

假设我们的系统需要在文件系统的指定位置生成一些数据文件（系统实现中可能存在多处这样的 位置），创建之前，我们需要首先检查这些指定位置是否存在，不存在则需要去创建它们。为了避免 不必要的代码散落，我们可以为系统中相应目标类提供一个Before Advice，对文件系统的指定路径进 行统一的检查或者初始化。代码清单9-6给出了用于初始化指定的资源路径的ResourceSetupBeforeAdvice定义。 

```
public class ResourceSetupBeforeAdvice implements MethodBeforeAdvice { 
 private Resource resource; 
 public ResourceSetupBeforeAdvice(Resource resource) 
 { 
 this.resource = resource; 
 } 
 public void before(Method method, Object[] args, Object target) 
 throws Throwable { 
 if(!resource.exists()) 
 { 
 FileUtils.forceMkdir(resource.getFile()); 
 } 
 } 
} 
```

#### ThrowsAdvice 

Spring中以接口定义org.springframework.aop.ThrowsAdvice对应通常AOP概念中的AfterThrowingAdvice。虽然该接口没有定义任何方法，但是在实现相应的ThrowsAdvice的时候，我们的 方法定义需要遵循如下规则： 

void afterThrowing([Method, args, target], ThrowableSubclass); 

其中，[]中的三个参数可以省略。我们可以根据将要拦截的Throwable的不同类型，在同一个 ThrowsAdvice中实现多个afterThrowing方法。框架将会使用Java反射机制（Java Reflection）来调 用这些方法 

```
public class ExceptionBarrierThrowsAdvice implements ThrowsAdvice { 
 
 public void afterThrowing(Throwable t) 
 { 
 // 普通异常处理逻辑
 } 
 
 public void afterThrowing(RuntimeException e) 
 { 
 // 运行时异常处理逻辑
 } 
 
 public void afterThrowing(Method m,Object[] args,Object target,ApplicationException e) 
 { 
 // 处理应用程序生成的异常
 } 
 ... 
} 
```



#### Around Advice 

Spring AOP没有提供After(Finally)Advice，使得我们没有一个合适的Advice类型来承载类似 于系统资源清除之类的横切逻辑。Spring AOP的AfterReturningAdvice不能更改Joinpoint所在方法 的返回值，使得我们在方法正常返回后无法对其进行更多干预。不过，有了Around Advice，这些问题 就都不是问题了。 

Spring中没有直接定义对应Around Advice的实现接口，而是直接采用AOP Alliance的标准接口， 即org.aopalliance.intercept.MethodInterceptor，该接口定义如下： 

```
public interface MethodInterceptor extends Interceptor { 
 Object invoke(MethodInvocation invocation) throws Throwable; 
} 
```

为了进一步演示MethodInterceptor的使用，我们可以设想这样的场景： 如果某个销售系统规 定，在商场优惠期间，所有商品一律8折出售（或者其他折扣条件），那么我们就应该在系统中所有 取得商品价格的位置插入这样的横切逻辑： 

```
Public class DiscountMethodInterceptor implements MethodInterceptor { 
 private static final Integer DEFAULT_DISCOUNT_RATIO = 80; 
 private static final IntRange RATIO_RANGE = new IntRange(5,95); 
 
 private Integer discountRatio = DEFAULT_DISCOUNT_RATIO; 
 private boolean campaignAvailable; 
 
 public Object invoke(MethodInvocation invocation) throws Throwable { 
 Object returnValue = invocation.proceed(); 
 if(RATIO_RANGE.containsInteger(getDiscountRatio()) && isCampaignAvailable()) 
 { 
 return ((Integer)returnValue)*getDiscountRatio()/100; 
 } 
 return returnValue; 
 } 
    
    
 private boolean isCampaignAvailable() { 
 return campaignAvailable; 
 }
 public void setCampaignAvailable(boolean campaignAvailable) { 
 this.campaignAvailable = campaignAvailable; 
 } 
 public Integer getDiscountRatio() { 
 return discountRatio; 
 } 
 public void setDiscountRatio(Integer discountRatio) { 
 this.discountRatio = discountRatio; 
 } 
} 
```

通过MethodInterceptor的invoke方法的MethodInvocation参数，我们可以控制对相应 Joinpoint的拦截行为。通过调用MethodInvocation的proceed()方法，可以让程序执行继续沿着调用链传播，这是通常我们所希望的行为。如果我们在哪一个MethodInterceptor中没有调用proceed()， 那么程序的执行将会在当前MethodInterceptor处“短路”，Joinpoint上的调用链将被中断，同一 Joinpoint上的其他MethodInterceptor的逻辑以及Joinpoint处的方法逻辑将不会被执行。除非你真的 知道自己在做什么，否则，不要忘记调用proceed()方法哦！

另外，我们还可以通过MethodInvocation 对象取得Joinpoint的更多信息。 正如在PerformanceMethodInterceptor中所看到的那样，我们可以在proceed()方法，也就是 Joinpoint处的逻辑执行之前或者之后插入相应的逻辑，甚至捕获proceed()方法可能抛出的异常。到现在，所以MethodInterceptor可以完成其他类型Advice可以完成的任务 





