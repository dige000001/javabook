## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

Spring提供了两种容器类型：BeanFactory和ApplicationContext。 

**BeanFactory**：基础类型IoC容器，提供完整的IoC服务支持。如果没有特殊指定，默认采用延迟初始化策略（lazy-load）。只有当客户端对象需要访问容器中的某个受管对象的时候，才对该受管对象进行初始化以及依赖注入操作。所以，相对来说，容器启动初期速度较快，所需要的资源有限。对于资源有限，并且功能要求不是很严格的场景，BeanFactory是比较合适的 IoC容器选择。 

**ApplicationContext：** ApplicationContext在BeanFactory的基础上构建，是相对比较高级的容器实现，除了拥有BeanFactory的所有支持，ApplicationContext还提供了其他高级特性，比如事件发布、国际化信息支持等，这些会在后面详述。ApplicationContext所管理的对象，在该类型容器启动之后，默认全部初始化并绑定完成。所以，相对于BeanFactory 来说，ApplicationContext要求更多的系统资源，同时，因为在启动时就完成所有初始化， 容器启动时间较之BeanFactory也会长一些。在那些系统资源充足，并且要求更多功能的场景中，ApplicationContext类型的容器是比较合适的选择。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e39ac2b5e2046e6b7409593da1cd0bf~tplv-k3u1fbpfcp-zoom-1.image)

# BeanFactory


**BeanFactory的定义**

```
public interface BeanFactory { 
 String FACTORY_BEAN_PREFIX = "&"; 
 Object getBean(String name) throws BeansException; 
 Object getBean(String name, Class requiredType) throws BeansException; 
 /** 
 * @since 2.5 
 */ 
 Object getBean(String name, Object[] args) throws BeansException; 
 boolean containsBean(String name); 
 boolean isSingleton(String name) throws NoSuchBeanDefinitionException; 
 /** 
 * @since 2.0.3 
 */ 
 boolean isPrototype(String name) throws NoSuchBeanDefinitionException; 
 /** 
 * @since 2.0.1 
 */ 
 boolean isTypeMatch(String name, Class targetType) throws NoSuchBeanDefinitionException; 
 Class getType(String name) throws NoSuchBeanDefinitionException; 
 String[] getAliases(String name); 
} 
```

上面代码中的方法基本上都是查询相关的方法，例如，取得某个对象的方法（getBean）、查询某个对象是否存在于容器中的方法（containsBean），或者取得某个bean的状态或者类型的方法等。 因为通常情况下，**对于独立的应用程序，只有主入口类才会跟容器的API直接耦合。**

****

## BeanFactory 的对象注册与依赖绑定方式 


**底层编码原理：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0d7f89ba9184fa1878ab0b50b2b28a4~tplv-k3u1fbpfcp-zoom-1.image)

每一个受管的对象，在容器中都会有一个BeanDefinition的实例（instance）与之相对应，该 BeanDefinition的实例负责保存对象的所有必要信息，包括其对应的对象的class类型、是否是抽象类、构造方法参数以及其他属性等。 在容器创建过程中，BeanDefinitionRegistry会加载注入对象和被注入对象的BeanDefinition实例（在通过配置文件加载Bean信息的方式中，这由BeanDefinitionReader实现类来完成），并设置相应的依赖关系，之后我们就可以使用BeanFactory的getBean等方法获取对象实例了。

下面例子演示了直接编码方式，通过Properties配置文件，通过Xml配置文件完成对象注册和依赖绑定的过程

### 直接编码

```
public static void main(String[] args) 
{ 
 DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory(); 
 BeanFactory container = (BeanFactory)bindViaCode(beanRegistry); 
 FXNewsProvider newsProvider = ➥
 (FXNewsProvider)container.getBean("djNewsProvider"); 
 newsProvider.getAndPersistNews(); 
} 
public static BeanFactory bindViaCode(BeanDefinitionRegistry registry) 
{ 
 AbstractBeanDefinition newsProvider = ➥
 new RootBeanDefinition(FXNewsProvider.class,true); 
 AbstractBeanDefinition newsListener = ➥
 new RootBeanDefinition(DowJonesNewsListener.class,true); 
 AbstractBeanDefinition newsPersister = ➥
 new RootBeanDefinition(DowJonesNewsPersister.class,true); 
 // 将bean定义注册到容器中
 registry.registerBeanDefinition("djNewsProvider", newsProvider); 
 registry.registerBeanDefinition("djListener", newsListener); 
 registry.registerBeanDefinition("djPersister", newsPersister); 
// 指定依赖关系
// 1. 可以通过构造方法注入方式
 ConstructorArgumentValues argValues = new ConstructorArgumentValues(); 
 argValues.addIndexedArgumentValue(0, newsListener); 
 argValues.addIndexedArgumentValue(1, newsPersister); 
 newsProvider.setConstructorArgumentValues(argValues); 
 // 2. 或者通过setter方法注入方式
 MutablePropertyValues propertyValues = new MutablePropertyValues(); 
 propertyValues.addPropertyValue(new ropertyValue("newsListener",newsListener)); 
 propertyValues.addPropertyValue(new PropertyValue("newPersistener",newsPersister)); 
 newsProvider.setPropertyValues(propertyValues); 
 // 绑定完成
 return (BeanFactory)registry; 
} 
```

### 通过Properties配置文件

```
public static void main(String[] args) 
{ 
 DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory(); 
 BeanFactory container = (BeanFactory)bindViaPropertiesFile(beanRegistry); 
 FXNewsProvider newsProvider = 
(FXNewsProvider)container.getBean("djNewsProvider"); 
 newsProvider.getAndPersistNews(); 
} 
public static BeanFactory bindViaPropertiesFile(BeanDefinitionRegistry registry) 
{ 
 PropertiesBeanDefinitionReader reader = 
new PropertiesBeanDefinitionReader(registry); 
    //读取配置文件，加载BeanDefinition实例
 reader.loadBeanDefinitions("classpath:../../binding-config.properties"); 
 return (BeanFactory)registry; 
} 
```

### 通过XML配置文件（重点）

```
public static void main(String[] args) 
{ 
 DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory(); 
 BeanFactory container = (BeanFactory)bindViaXMLFile(beanRegistry); 
 FXNewsProvider newsProvider = ➥
 (FXNewsProvider)container.getBean("djNewsProvider"); 
 newsProvider.getAndPersistNews(); 
} 
public static BeanFactory bindViaXMLFile(BeanDefinitionRegistry registry) 
{ 
 XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(registry); 
 reader.loadBeanDefinitions("classpath:../news-config.xml"); 
 return (BeanFactory)registry; 
 // 或者直接
 //return new XmlBeanFactory(new ClassPathResource("../news-config.xml")); 
} 
```

与为Properties配置文件格式提供PropertiesBeanDefinitionReader相对应，Spring同样为XML 格式的配置文件提供了现成的BeanDefinitionReader实现，即XmlBeanDefinitionReader。 XmlBeanDefinitionReader负责读取Spring指定格式的XML配置文件并解析，之后将解析后的文件内容映射到相应的BeanDefinition，并加载到相应的BeanDefinitionRegistry中（在这里是DefaultListableBeanFactory）。这时，整个BeanFactory就可以放给客户端使用了。除了提供XmlBeanDefinitionReader用于XML格式配置文件的加载，Spring还在DefaultListableBeanFactory的基础上构建了简化XML格式配置加载的XmlBeanFactory实现。从以上代码最后注释掉的一行，你可以看到使用了XmlBeanFactory之后，完成XML的加载和BeanFactory的初始化是多么简单。

## 容器背后的秘密 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/958bc3aeec794b63bf9014329712c3cf~tplv-k3u1fbpfcp-zoom-1.image)

### 1.BeanFactoryPostProcessor

Spring提供了一种叫做BeanFactoryPostProcessor的容器扩展机制。该机制允许我们在容器实例化相应对象之前，对注册到容器的BeanDefinition所保存的信息做相应的修改。这就相当于在容 器实现的第一阶段最后加入一道工序，让我们对最终的BeanDefinition做一些额外的操作，比如修改其中bean定义的某些属性，为bean定义增加其他信息等。 

Spring已经提供了几个现成的BeanFactoryPostProcessor实现类，所以，大多时候我们很少自己去实现某个BeanFactoryPostProcessor。其中，

org.springframework.beans.factory.config.**PropertyPlaceholderConfigurer**

和org.springframework.beans.factory. config.**PropertyOverrideConfigurer**

是两个比较常用的BeanFactoryPostProcessor。 

我们可以通过两种方式来应用 BeanFactoryPostProcessor， 分别针对基本的 BeanFactory和较为先进的ApplicationContext。 

对于BeanFactory来说，我们需要用手动方式应用所有的BeanFactoryPostProcessor 

```
// 声明将被后处理的BeanFactory实例
ConfigurableListableBeanFactory beanFactory = 
    new XmlBeanFactory(new ClassPathResource("xml配置文件")); 
// 声明要使用的BeanFactoryPostProcessor 
PropertyPlaceholderConfigurer propertyPostProcessor = new PropertyPlaceholderConfigurer(); 
propertyPostProcessor.setLocation(new ClassPathResource("后处理文件，比如jdbc连接信息：jdbc.properties")); 
// 执行后处理操作
propertyPostProcessor.postProcessBeanFactory(beanFactory);
```

对于配置在配置文件的BeanFactoryPostProcessor，也可以通过getbean方法获取

```
public class PropertyconfigurerDemo {
public static void main(String[]args) {
	ConfigurableListableBeanFactory bf= new XmlBeanFactory(new ClassPathResource
	("/META--INF/BeanFactory.xml") ) ;
	BeanFactoryPostProcessor bfpp=(BeanFactoryPostProcessor) bf.getBean ("bfpp");
    bfpp.postProcessBeanFactory (bf);
	system.out.println (bf.getBean ("simpleBean") ) ;
)
```

对于ApplicationContext来说，情况看起来要好得多。因为ApplicationContext会自动识别配置文件中的BeanFactoryPostProcessor并应用它，所以，相对于BeanFactory，在ApplicationContext 中加载并应用BeanFactoryPostProcessor，仅需要在XML配置文件中将这些BeanFactoryPostProcessor简单配置一下即可。 

```
<beans> 
 <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer"> 
 <property name="locations"> 
 <list> 
 <value>conf/jdbc.properties</value> 
 <value>conf/mail.properties</value> 
 </list> 
 </property> 
 </bean> 
 ... 
</beans> 
```

#### PropertyPlaceholderConfigurer 

通常情况下，我们不想将类似于系统管理相关的信息同业务对象相关的配置信息混杂到XML配置 文件中，以免部署或者维护期间因为改动繁杂的XML配置文件而出现问题。**我们会将一些数据库连接信息、邮件服务器等相关信息单独配置到一个properties文件中**，这样，如果因系统资源变动的话，只需要关注这些简单properties配置文件即可。 

PropertyPlaceholderConfigurer允许我们在XML配置文件中使用占位符（PlaceHolder）， 并将这些占位符所代表的资源单独配置到简单的properties文件中来加载。以数据源的配置为例，使用了PropertyPlaceholderConfigurer之后，可以在XML配置文件中按照以下代码的方式配置数据源，而不用将连接地址、用户名和密码等都配置到 XML中 

```
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" ➥
destroy-method="close"> 
 <property name="url"> 
 <value>${jdbc.url}</value> 
 </property> 
 <property name="driverClassName"> 
 <value>${jdbc.driver}</value> 
 </property> 
 <property name="username"> 
 <value>${jdbc.username}</value> 
 </property> 
 <property name="password"> 
 <value>${jdbc.password}</value> 
 </property> 
 <property name="maxActive"> 
 <value>100</value> 
 </property> 
  ...
  </bean> 
```

基本机制就是之前所说的那样。当BeanFactory在第一阶段加载完成所有配置信息时，BeanFactory中保存的对象的属性信息还只是以占位符的形式存在，如${jdbc.url}、${jdbc.driver}。当 PropertyPlaceholderConfigurer作为BeanFactoryPostProcessor被应用时，它会使用properties 配置文件中的配置信息来替换相应BeanDefinition中占位符所表示的属性值。这样，当进入容器实 现的第二阶段实例化bean时，bean定义中的属性值就是最终替换完成的了 。

#### PropertyOverrideConfigurer 

可以通过PropertyOverrideConfigurer对容器中配置的任何你想处理的bean定义的property信 息进行覆盖替换。比如之前的dataSource定义中， maxActive的值为100，如果我们觉得100不合适，那么可以通过PropertyOverrideConfigurer在其相应的properties文件中做如下所示配置，把100这个值给覆盖掉，如将其配置为200： 

dataSource.maxActive=200 

**properties文件中的键是以XML中配置的bean定义的beanName为标志开始的（通常就是id指定的值），后面跟着相应被覆盖的property的名称，比如上面的maxActive。**

****

#### CustomEditorConfigurer 

其他两个BeanFactoryPostProcessor都是通过对BeanDefinition中的数据进行变更以达到某种目的。与它们有所不同，CustomEditorConfigurer是另一种类型的BeanFactoryPostProcessor 实现，它只是辅助性地将后期会用到的信息注册到容器，对BeanDefinition没有做任何变动。 

XML所记载的，都是String 类型，即容器从XML格式的文件中读取的都是字符串形式，最终应用程序却是由各种类型的对象所构成。**要想完成这种由字符串到具体对象的转换（不管这个转换工作最终由谁来做），都需要这种转换规则相关的信息，而CustomEditorConfigurer就是帮助我们传达类似信息的。** Spring内部有大多数类型的 PropertyEditor，来帮助从字符串到其他类型的转换，但对于我们自定义的转换，则需要我们给出针对这种类型的PropertyEditor实现 。

具体的例子看《Spring揭秘》P72 （Real P86）

### 2.Bean的一生

我们说，ApplicationContext在容器初始化时就会实例化所有bean，而这是通过在初始化阶段，调用所有getbean()实现的，**也就是说，对于一个bean，只有在第一次getbean()调用时，才会实例化对象**，之后的getbean()都只是返回该对象的缓存（prototype除外）。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d53a2c9bb3bc444591738a04cbe11fdb~tplv-k3u1fbpfcp-zoom-1.image)

#### 1.Bean的实例化与BeanWrapper 

容器在内部实现的时候，采用“策略模式（Strategy Pattern）”来决定采用何种方式初始化bean实例。 通常，可以通过**反射**或者**CGLIB动态字节码生成**来初始化相应的bean实例或者动态生成其子类。 

org.springframework.beans.factory.support.InstantiationStrategy定义是实例化策略的抽象接口，其直接子类SimpleInstantiationStrategy实现了简单的对象实例化功能，可以通过反射来实例化对象实例，但不支持方法注入方式的对象实例化。 

CglibSubclassingInstantiationStrategy继承了SimpleInstantiationStrategy的以反射方式实例化对象的功能，并且通过CGLIB 的动态字节码生成功能，该策略实现类可以动态生成某个类的子类，进而满足了方法注入所需的对象 实例化需求。默认情况下，容器内部采用的是CglibSubclassingInstantiationStrategy

容器只要根据相应bean定义的BeanDefintion取得实例化信息，结合CglibSubclassingInstantiationStrategy以及不同的bean定义类型，就可以返回实例化完成的对象实例。但是，返回方式上有些“点缀”。**不是直接返回构造完成的对象实例，而是以BeanWrapper对构造完成的对象实例进行包裹，返回相应的BeanWrapper实例。**

至此，第一步结束。

BeanWrapper接口通常在Spring框架内部使用，它有一个实现类org.springframework.beans. BeanWrapperImpl。**其作用是对某个bean进行“包裹”，然后对这个“包裹”的bean进行操作，比如设置或者获取bean的相应属性值。而在第一步结束后返回BeanWrapper实例而不是原先的对象实例， 就是为了第二步“设置对象属性”。**  BeanWrapper定义继承了org.springframework.beans.PropertyAccessor接口，可以以统一的方式对对象属性进行访问；BeanWrapper定义同时又直接或者间接继承了PropertyEditorRegistry 和TypeConverter接口。 **在第一步构造完成对象之后，Spring会根据对象实例构造一个BeanWrapperImpl实例，然后将之前CustomEditorConfigurer注册的PropertyEditor复制一份给BeanWrapperImpl实例（这就是BeanWrapper同时又 是PropertyEditorRegistry的原因）。这样，当BeanWrapper转换类型、设置对象属性值时，就不会无从下手了**。




以下展示用BeanWrapper操作对象

```
Object provider = Class.forName("package.name.FXNewsProvider").newInstance(); 
Object listener = Class.forName("package.name.DowJonesNewsListener").newInstance(); 
Object persister = Class.forName("package.name.DowJonesNewsPersister").newInstance(); 

BeanWrapper newsProvider = new BeanWrapperImpl(provider); 
newsProvider.setPropertyValue("newsListener", listener); 
newsProvider.setPropertyValue("newPersistener", persister); 

assertTrue(newsProvider.getWrappedInstance() instanceof FXNewsProvider); 
assertSame(provider, newsProvider.getWrappedInstance()); 
assertSame(listener, newsProvider.getPropertyValue("newsListener")); 
assertSame(persister, newsProvider.getPropertyValue("newPersistener")); 
```

#### 2.各种Aware接口

当对象实例化完成并且相关属性以及依赖设置完成之后，Spring容器会检查当前对象实例是否实现了一系列的以Aware命名结尾的接口定义。如果是，则将这些Aware接口定义中规定的依赖注入给 当前对象实例。 

#### 3.BeanPostProcessor 

如果我们想在Spring容器中完成bean实例化、配置以及其他初始化方法前后要添加一些自己逻辑处理。我们需要定义一个或多个BeanPostProcessor接口实现类，然后注册到Spring IoC容器中。

除了检查标记接口以便应用自定义逻辑，还可以通过BeanPostProcessor对当前对象实例做更多 的处理。比如替换当前对象实例或者字节码增强当前对象实例等。Spring的AOP则更多地使用 BeanPostProcessor来为对象生成相应的代理对象，如org.springframework.aop.framework. autoproxy.BeanNameAutoProxyCreator。

**例子：**

假设系统中所有的IFXNewsListener实现类需要从某个位置取得相应的服务器连接密码，而且系统中保存的密码是加密的，那么在IFXNewsListener发送这个密码给新闻服务器进行连接验证的时候，首先需要对系统中取得的密码进行解密，然后才能发送。我们将采用BeanPostProcessor技术，对所有的IFXNewsListener的实现类进行统一的解密操作。 

(1) 标注需要进行解密的实现类 

```
public interface PasswordDecodable { 
	 String getEncodedPassword(); 
 	 void setDecodedPassword(String password); 
} 
public class DowJonesNewsListener implements IFXNewsListener,PasswordDecodable { 
 private String password; 
 
 public String[] getAvailableNewsIds() { 
 // 省略
 } 
 public FXNewsBean getNewsByPK(String newsId) { 
 // 省略
 } 
 public void postProcessIfNecessary(String newsId) { 
 // 省略
 } 
 public String getEncodedPassword() { 
 return this.password; 
 } 
 public void setDecodedPassword(String password) { 
 this.password = password; 
 } 
}
```

(2) 实现相应的BeanPostProcessor对符合条件的Bean实例进行处理 

我们通过PasswordDecodable接口声明来区分将要处理的对象实例，当检查到当前对象实例实 现了该接口之后，就会从当前对象实例取得加密后的密码，并对其解密。然后将解密后的密码设置回 当前对象实例。之后，返回的对象实例所持有的就是解密后的密码，逻辑如下所示

```
public class PasswordDecodePostProcessor implements BeanPostProcessor { 
 public Object postProcessAfterInitialization(Object object, String beanName) 
  throws BeansException { 
 	return object; 
 } 
 public Object postProcessBeforeInitialization(Object object, String beanName) 
 throws BeansException { 
 	if(object instanceof PasswordDecodable) { 
 		String encodedPassword = ((PasswordDecodable)object).getEncodedPassword(); 
 		String decodedPassword = decodePassword(encodedPassword); 
 		((PasswordDecodable)object).setDecodedPassword(decodedPassword); 
 	} 
 	return object; 
 } 
 private String decodePassword(String encodedPassword) { 
 // 实现解码逻辑
 return encodedPassword; 
 } 
} 
```

(3) 将自定义的BeanPostProcessor注册到容器 

只有将自定义的BeanPostProcessor实现类告知容器，容器才会在合适的时机应用它。所以，我们需要将PasswordDecodePostProcessor注册到容器。

对于BeanFactory类型的容器来说，我们需要通过手工编码的方式将相应的BeanPostProcessor 注册到容器，也就是调用ConfigurableBeanFactory的addBeanPostProcessor()方法，见如下代码： 

```
ConfigurableBeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource(...)); 
beanFactory.addBeanPostProcessor(new PasswordDecodePostProcessor()); 
... 
// getBean();
```

对于ApplicationContext容器来说，事情则方便得多，直接将相应的BeanPostProcessor实现类通过通常的XML配置文件配置一下即可。ApplicationContext容器会自动识别并加载注册到容器 的BeanPostProcessor，如下配置内容将我们的PasswordDecodePostProcessor注册到容器： 

```
<beans> 
 <bean id="passwordDecodePostProcessor" class="package.name.PasswordDecodePostProcessor"> 
 <!--如果需要，注入必要的依赖--> 
 </bean> 
 ... 
</beans> 
```

合理利用BeanPostProcessor这种Spring的容器扩展机制，将可以构造强大而灵活的应用系统

#### 4.InitializingBean和init-method （例子看“使用”部分）

org.springframework.beans.factory.InitializingBean是容器内部使用的一个对象生命周期标识接口，其定义如下： 

```
public interface InitializingBean { 
 void afterPropertiesSet() throws Exception; 
} 
```

该接口定义很简单，其作用在于，在对象实例化过程调用过“BeanPostProcessor的前置处理” 之后，会接着检测当前对象是否实现了InitializingBean接口，如果是，则会调用其afterPropertiesSet()方法进一步调整对象实例的状态。比如，在有些情况下，某个业务对象实例化完成后，还 不能处于可以使用状态。这个时候就可以让该业务对象实现该接口，并在方法afterPropertiesSet() 中完成对该业务对象的后续处理。 虽然该接口在Spring容器内部广泛使用，但如果真的让我们的业务对象实现这个接口，则显得 Spring容器比较具有侵入性。所以，**Spring还提供了另一种方式来指定自定义的对象初始化操作，那就是在XML配置的时候，使用的init-method属性。 通过init-method，系统中业务对象的自定义初始化操作可以以任何方式命名，而不再受制于 InitializingBean的afterPropertiesSet()。**

****

#### 5. DisposableBean与destroy-method（例子看“使用”部分）

当所有的一切，该设置的设置，该注入的注入，该调用的调用完成之后，容器将检查singleton类 型的bean实例，看其是否实现了org.springframework.beans.factory.DisposableBean接口。或 者其对应的bean定义是否通过的destroy-method属性指定了自定义的对象销毁方法。如果是， 就会为该实例注册一个用于对象销毁的回调（Callback），以便在这些singleton类型的对象实例销毁之前，执行销毁逻辑。 与InitializingBean和init-method用于对象的自定义初始化相对应，DisposableBean和 destroy-method为对象提供了执行自定义销毁逻辑的机会。 最常见到的该功能的使用场景就是在Spring容器中注册数据库连接池，在系统退出后，连接池应该关闭，以释放相应资源。 

如果spring不在Servlet或者EJB容器中，就需要我们手动关闭容器以调用该方法

```
public class ApplicationLauncher 
{ 
 public static void main(String[] args) { 
 BasicConfigurator.configure(); 
 BeanFactory container = new XmlBeanFactory(new ClassPathResource("...")); 
 BusinessObject bean = (BusinessObject)container.getBean("..."); 
 bean.doSth(); 
((ConfigurableListableBeanFactory)container).destroySingletons(); 
 // 应用程序退出，容器关闭
 } 
} 
```

如果不能在合适的时机调用destroySingletons()，那么所有实现了DisposableBean接口的对 象实例或者声明了destroy-method的bean定义对应的对象实例，它们的自定义对象销毁逻辑就形同 虚设，因为根本就不会被执行！ 

对于ApplicationContext容器来说。道理是一样的。但AbstractApplicationContext为我们提供了registerShutdownHook()方法，该方法底层使用标准的Runtime类的addShutdownHook()方式来调用相应bean对象的销毁逻辑，从而保证在Java虚拟机退出之前，这些singtleton类型的bean对象 实例的自定义销毁逻辑会被执行。当然AbstractApplicationContext注册的shutdownHook不只是调用对象实例的自定义销毁逻辑，也包括ApplicationContext相关的事件发布等

```
public class ApplicationLauncher 
{ 
 public static void main(String[] args) { 
 BasicConfigurator.configure(); 
 BeanFactory container = new ClassPathXmlApplicationContext("..."); 
 ((AbstractApplicationContext)container).registerShutdownHook(); 
 BusinessObject bean = (BusinessObject)container.getBean("..."); 
 bean.doSth(); 
 // 应用程序退出，容器关闭
 } 
} 
```

# ApplicationContext

## 统一资源加载策略

### Spring中的Resource

Spring框架内部使用org.springframework.core.io.Resource接口作为所有资源的抽象和访 问接口，我们之前在构造BeanFactory的时候已经接触过它，如下代码： 

BeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("...")); 

其中ClassPathResource就是Resource的一个特定类型的实现，代表的是位于Classpath中的资源。 Resource接口可以根据资源的不同类型，或者资源所处的不同场合，给出相应的具体实现。Spring框架在这个理念的基础上，提供了一些实现类（可以在org.springframework.core.io包下找到这 些实现类）： 

-   **ByteArrayResource**。将字节（byte）数组提供的数据作为一种资源进行封装，如果通过 InputStream形式访问该类型的资源，该实现会根据字节数组的数据，构造相应的ByteArrayInputStream并返回。
-   **ClassPathResource**。该实现从Java应用程序的ClassPath中加载具体资源并进行封装，可以使用指定的类加载器（ClassLoader）或者给定的类进行资源加载。  

<!---->

-   **FileSystemResource**。对java.io.File类型的封装，所以，我们可以以文件或者URL的形式对该类型资源进行访问，只要能跟File打的交道，基本上跟FileSystemResource也可以。 
-   **UrlResource**。通过java.net.URL进行的具体资源查找定位的实现类，内部委派URL进行具体的资源操作。

<!---->

-   **InputStreamResource**。将给定的InputStream视为一种资源的Resource实现类，较为少用。可能的情况下，以ByteArrayResource以及其他形式资源实现代之。 


### ResourceLoader

ResourceLoader可以根据给定的资源位置定位资源。 ResourceLoader定义如下： 

```
public interface ResourceLoader { 
 String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX; 
 Resource getResource(String location); 
 ClassLoader getClassLoader(); 
}
```

其中最主要的就是Resource getResource(String location);方法，通过它，我们就可以根 据指定的资源位置，定位到具体的资源实例。 

#### DefaultResourceLoader 

ResourceLoader有一个默认的实现类，即org.springframework.core.io.DefaultResourceLoader，该类默认的资源查找处理逻辑如下：

(1) 首先检查资源路径是否以classpath:前缀打头，如果是，则尝试构造ClassPathResource类型资源并返回。

(2) 否则，(a) 尝试通过URL，根据资源路径来定位资源，如果没有抛出MalformedURLException， 则会构造UrlResource类型的资源并返回；(b)如果还是无法根据资源路径定位指定的资源，则委派 getResourceByPath(String)方法来定位，DefaultResourceLoader的getResourceByPath(String) 方法默认实现逻辑是，构造ClassPathResource类型的资源并返回。 

```
ResourceLoader resourceLoader = new DefaultResourceLoader(); 
Resource fakeFileResource = resourceLoader.getResource("D:/spring21site/README"); 
assertTrue(fakeFileResource instanceof ClassPathResource); 
assertFalse(fakeFileResource.exists());//为啥呢？
 
Resource urlResource1 = resourceLoader.getResource("file:D:/spring21site/README"); 
assertTrue(urlResource1 instanceof UrlResource); //返回的是UrlResource类型
 
Resource urlResource2 = resourceLoader.getResource("http://www.spring21.cn"); 
assertTrue(urlResource2 instanceof UrlResource);
```




#### FileSystemResourceLoader 

为了避免DefaultResourceLoader在最后getResourceByPath(String)方法上的不恰当处理， 我们可以使用org.springframework.core.io.FileSystemResourceLoader，它继承自DefaultResourceLoader，但覆写了getResourceByPath(String)方法，使之从文件系统加载资源并以 FileSystemResource类型返回。这样，我们就可以取得预想的资源类型。 

```
public void testResourceTypesWithFileSystemResourceLoader() 
{ 
 ResourceLoader resourceLoader = new FileSystemResourceLoader(); 
 Resource fileResource = resourceLoader.getResource("D:/spring21site/README"); 
 assertTrue(fileResource instanceof FileSystemResource); 
 assertTrue(fileResource.exists()); //能取得到资源
 
 Resource urlResource = resourceLoader.getResource("file:D:/spring21site/README"); 
 assertTrue(urlResource instanceof UrlResource); //为啥返回的不是FileSystemResource?
}
```

### ResourcePatternResolver ——批量查找的ResourceLoader 

ResourcePatternResolver是ResourceLoader的扩展，ResourceLoader每次只能根据资源路径返回确定的单个Resource实例，而**ResourcePatternResolver则可以根据指定的资源路径匹配模式， 每次返回多个Resource实例。** 接口org.springframework.core.io.support.ResourcePatternResolver定义如下： 

```
public interface ResourcePatternResolver extends ResourceLoader { 
 String CLASSPATH_ALL_URL_PREFIX = "classpath*:"; 
 Resource[] getResources(String locationPattern) throws IOException; 
} 
```

ResourcePatternResolver在继承ResourceLoader原有定义的基础上，又引入了Resource[] getResources(String)方法定义，以支持根据路径匹配模式返回多个Resources的功能。它同时还 引入了一种新的协议前缀**classpath*** :，针对这一点的支持，将由相应的子类实现给出。 

ResourcePatternResolver最常用的一个实现是org.springframework.core.io.support. PathMatchingResourcePatternResolver，该实现类支持ResourceLoader级别的资源加载，支持基于Ant风格的路径匹配模式（类似于**/*.suffix之类的路径形式），支持ResourcePatternResolver 新增加的classpath*:前缀等，基本上集所有技能于一身。

在构造PathMatchingResourcePatternResolver实例的时候，可以指定一个ResourceLoader， 如果不指定的话，则PathMatchingResourcePatternResolver内部会默认构造一个DefaultResourceLoader实例。PathMatchingResourcePatternResolver内部会将匹配后确定的资源路径， 委派给它的ResourceLoader来查找和定位资源。这样，如果不指定任何ResourceLoader的话，PathMatchingResourcePatternResolver在加载资源的行为上会与DefaultResourceLoader基本相同， 只存在返回的Resource数量上的差异。如下代码表明了二者在资源加载行为上的一致性： 

```
ResourcePatternResolver resourceResolver = new PathMatchingResourcePatternResolver(); 
Resource fileResource = resourceResolver.getResource("D:/spring21site/README"); 
assertTrue(fileResource instanceof ClassPathResource); 
assertFalse(fileResource.exists()); 
```

使用FileSystemResourceLoader替换默认的DefaultResourceLoader，从而使得PathMatchingResourcePatternResolver的行为跟使用FileSystemResourceLoader一样：

```
public void testResourceTypesWithPathMatchingResourcePatternResolver() 
{ 
resourceResolver = new PathMatchingResourcePatternResolver(new FileSystemResourceLoader()); 
 fileResource = resourceResolver.getResource("D:/spring21site/README"); 
 assertTrue(fileResource instanceof FileSystemResource); 
 assertTrue(fileResource.exists()); 
}
```

即，**ResourcePatternResolver获取资源的方式与其Resourceloader一致**

****

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3eae5f1a969d4fa586887b862aacd166~tplv-k3u1fbpfcp-zoom-1.image)

### ApplicationContext 与 ResourceLoader 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/230abe04e69c4455b3816d7fdc982528~tplv-k3u1fbpfcp-zoom-1.image)

由上图， ApplicationContext继承了ResourcePatternResolver, 所以，任何的ApplicationContext实现都可以看作是一个 ResourceLoader甚至ResourcePatternResolver。而这就是ApplicationContext支持Spring内统一资源加载策略的真相。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc61a85a6c1143b4b9cecc247f25b170~tplv-k3u1fbpfcp-zoom-1.image)

所有的ApplicationContext实现类都间接或直接继承自AbstractApplicationContext，而AbstractApplicationContext继承了DefaultResourceLoader，内部又声明了一个PathMatchingResourcePatternResolver（其内的ResourceLoader就是DefaultResourceLoader），故可以实现对资源统一加载。

```
//把ApplicationContext当作ResourceLoader来使用

ResourceLoader resourceLoader = new ClassPathXmlApplicationContext("配置文件路径"); 
// 或者
// ResourceLoader resourceLoader = new FileSystemXmlApplicationContext("配置文件路径"); 
Resource fileResource = resourceLoader.getResource("D:/spring21site/README"); 
assertTrue(fileResource instanceof ClassPathResource); 
assertFalse(fileResource.exists()); 
Resource urlResource2 = resourceLoader.getResource("http://www.spring21.cn"); 
assertTrue(urlResource2 instanceof UrlResource); 
```




#### 奇怪的应用

如果某个bean需要依赖于ResourceLoader来查找定位资源，我们可以为其注入容器中声明的某个具体的ResourceLoader实现，该bean也无需实现任何接口，直接通过构造方法 注入或者setter方法注入规则声明依赖即可，这样处理是比较合理的。 不过，如果你不介意你的bean定 义依赖于Spring的API，那不妨考虑用一下Spring提供的便利： ApplicationContext容器本身就是一个ResourceLoader，我们为了该类还需要单独提供 一个resourceLoader实例就有些多余了，直接将当前的ApplicationContext容器作为ResourceLoader注入不就行了？而ResourceLoaderAware和ApplicationContextAware接口正好可以帮助我 们做到这一点，只不过现在的FooBar需要依赖于Spring的API了。不过，在我看来，这没有什么大不 了，因为我们从来也没有真正逃脱过依赖（这种依赖也好，那种依赖也罢）。 

```
public class FooBar implements ResourceLoaderAware{ 
 private ResourceLoader resourceLoader; 
 
 public void foo(String location) 
 { 
 System.out.println(getResourceLoader().getResource(location).getClass()); 
} 
    
 public ResourceLoader getResourceLoader() { 
 return resourceLoader; 
 } 
    
 public void setResourceLoader(ResourceLoader resourceLoader) { 
 this.resourceLoader = resourceLoader; 
 } 
} 


public class FooBar implements ApplicationContextAware{ 
 private ResourceLoader resourceLoader; 
 
 public void foo(String location) 
 { 
 System.out.println(getResourceLoader().getResource(location).getClass()); 
 } 
 public ResourceLoader getResourceLoader() { 
 return resourceLoader; 
 } 
 public void setApplicationContext(ApplicationContext ctx) ➥
 throws BeansException { 
 this.resourceLoader = ctx; 
 } 
} 
```

```
<bean id="fooBar" class="...FooBar"> 
</bean> 
```

容器启动的时候，就会自动将当前ApplicationContext容器本身 注入到FooBar中，因为ApplicationContext类型容器可以自动识别Aware接口。 当然，如果应用场景仅使用ResourceLoader类型即可满足需求，那么，还是使用ResourceLoaderAware比较合适，ApplicationContextAware相对来说过于宽泛了些（当然，使用也未尝不可）

### Resource类型的注入

我们之前讲过，容器可以将bean定义文件中的字符串形式表达的信息，正确地转换成具体对象定 义的依赖类型。对于那些Spring容器提供的默认的PropertyEditors无法识别的对象类型，我们可以提供自定义的PropertyEditor实现并注册到容器中，以供容器做类型转换的时候使用。 

默认情况下， BeanFactory容器不会为org.springframework.core.io.Resource类型提供相应的PropertyEditor，所以，如果我们想注入Resource类型的bean定义，就需要注册自定义的PropertyEditor到 BeanFactory容器。 不过，对于ApplicationContext来说，我们无需这么做，**因为ApplicationContext容器可以正确识别Resource类型并转换后注入相关对象。**

假设有一个XMailer类，它依赖于一个模板来提供邮件发送的内容，我们声明模板为Resource类型，那么，最终的XMailer定义也就如以下所示：

```
public class XMailer { 
 private Resource template; 
 public void sendMail(Map mailCtx) 
 { 
 // String mailContext = merge(getTemplate().getInputStream(),mailCtx); 
 //... 
 } 
 public Resource getTemplate() { 
 return template; 
 } 
 public void setTemplate(Resource template) { 
 this.template = template; 
 } 
} 
```

我们只需要像普通bean那样注入Resource就行

```
<bean id="mailer" class="...XMailer"> 
 <property name="template" value="..resources.default_template.vm"/> 
</bean> 
```

ApplicationContext启动伊始，会通过一个org.springframework.beans.support.ResourceEditorRegistrar来注册Spring提供的针对Resource类型的PropertyEditor实现到容器中，这个 PropertyEditor叫做 org.springframework.core.io.ResourceEditor。这样， ApplicationContext就可以正确地识别Resource类型的依赖了。至于ResourceEditor怎么实现我就不用说了 吧？你想啊，把配置文件中的路径让ApplicationContext作为ResourceLoader给你定位一下不就得了。
### 在特定情况下，ApplicationContext的Resource加载行为 

两种方式指定资源定位方式：

```
// 代码中使用协议前缀
ResourceLoader resourceLoader = new ➥
FileSystemXmlApplicationContext("classpath:conf/container-conf.xml"); 
```

```
// 配置中使用协议前缀
<bean id="..." class="..."> 
 <property name="..."> 
 <value>classpath:resource/template.vm</value> 
 </property> 
</bean>
```

ClassPathXmlApplicationContext 默认从classpath:中定位资源

```
ApplicationContext ctx = new ClassPathXmlApplicationContext("conf/appContext.xml");
//效果等同于
ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:conf/appContext.xml");  
```

FileSystemXmlApplicationContext 默认从文件系统中定位资源，但若为资源位置显示声明classpath:，它就会从classpath中定位资源，返回的是ClassPathResource

## 国际化信息支持 

假设我们正在开发一个支持多国语言的Web应用程序,要求系统能够根据客户端的系统的语言类型返回对应的界面：英文的操作系统返回英文界面，而中文的操作系统则返回中文界面——这便是典型的i18n国际化问题。对于有国际化要求的应用系统,我们不能简单地采用硬编码的方式编写用户界面信息、报错信息等内容，而必须为这些需要国际化的信息进行特殊处理。简单来说，就是**为每种语言提供一套相应的资源文件**，并以规范化命名的方式保存在特定的目录中 **(这就是ResourceBundle所管理的资源文件)** ，由系统自动根据客户端语言选择适合的资源文件。

“国际化信息”也称为“本地化信息”，一般需要两个条件才可以确定一个特定类型的本地化信息，它们分别是“**语言类型**”和“**国家/地区的类型**”。如中文本地化信息既有中国大陆地区的中文，又有中国台湾地区、中国香港地区的中文，还有新加坡地区的中文。Java通过java.util.Locale类表示一个本地化对象，它允许通过语言参数和国家/地区参数创建一个确定的本地化对象。

### Locale

java.util.Locale是表示语言和国家/地区信息的本地化类，它是创建国际化应用的基础。下面给出几个创建本地化对象的示例:

```
//带有语言和国家/地区信息的本地化对象
Locale locale1 = new Locale ( "zh" ,"CN") ;
//只有语言信息的本地化对象
Locale locale2 = new Locale ("zh" );
//等同于Locale("zh", "CN")
Locale locale3 = Locale.CHINA;
//等同于Locale("zh")
Locale locale4 = Locale.CHINESE;
//获取本地系统默认的本地化对象
Locale locale 5= Locale.getDefault();
```

JDK 的 java.util包中提供了几个支持本地化的格式化操作工具类:NumberFormat 、DateFormat、MessageFormat，而在Spring 中的国际化资源操作也无非是对于这些类的封装操作，我们仅仅介绍下MessageFormat的用法以帮助大家回顾:

```
//①信息格式化串
String pattern1 = "{0}，你好!你于{1}在工商银行存入{2}元。";
String pattern2 = "At (1,time , short} on{1,date,long}，{0} paid (2, number，currency) ." ;

//②用于动态替换占位符的参数
Object[ ] params =( "John",new GregorianCalendar().getTime (),1.0E3};
                   
//③使用默认本地化对象格式化信息
String msgl = MessageFormat.format (pattern1,params) ;
                   
//4使用指定的本地化对象格式化信息
MessageFormat mf = new MessageFormat (pattern2,Locale.tS);
String msg2 = mf.format (params ) ;
System. out.println (msgl) ;
System. out.println(msg2);
```

### ResourceBundle

ResourceBundle用来保存特定于某个Locale的信息（可以是String类型信息，也可以是任何类型 的对象）。通常，ResourceBundle管理一组信息序列，所有的信息序列有统一的一个basename，然后 特定的Locale的信息，可以根据basename后追加的语言或者地区代码来区分。比如，我们用一组 properties文件来分别保存不同国家地区的信息，可以像下面这样来命名相应的properties文件： 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0db86dc88aca4872afa2f11d2f9c104a~tplv-k3u1fbpfcp-zoom-1.image)

其中，文件名中的messages部分称作ResourceBundle将加载的资源的basename，其他语言或地 区的资源在basename的基础上追加Locale特定代码。 每个资源文件中都有相同的键来标志具体资源条目，但每个资源内部对应相同键的资源条目内 容，则根据Locale的不同而不同。如下代码片段演示了两个不同的资源文件内容的对比情况： 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34692dd7f72a4b429d2b61b128e10eba~tplv-k3u1fbpfcp-zoom-1.image)




### MessageSource

Spring定义了访问国际化信息的 MessageSource接口

```
public interface MessageSource { 
 String getMessage(String code, Object[] args, String defaultMessage, Locale locale); 
 String getMessage(String code, Object[] args, Locale locale) throws NoSuchMessageException; 
 String getMessage(MessageSourceResolvable resolvable, Locale locale) throws
NoSuchMessageException; 
} 
```

ApplicationContext除了实现了ResourceLoader以支持统一的资源加载，它还实现了MessageSource接口，那么就跟ApplicationContext因为实现了ResourceLoader而可以当作 ResourceLoader来使用一样，ApplicationContext现在也是一个MessageSource了 .

在默认情况下，ApplicationContext将委派容器中一个**名称为messageSource**的MessageSource接口实现来完成MessageSource应该完成的职责。如果找不到这样一个名字的MessageSource 实现，ApplicationContext内部会默认实例化一个不含任何内容的StaticMessageSource实例，以 保证相应的方法调用。所以通常情况下，如果要提供容器内的国际化信息支持，我们会添加如以下代码的配置信息到容器的配置文件中 

```
<beans> 
 <bean id="messageSource" //ID必须为这个
       class="org.springframework.context.support.ResourceBundleMessageSource"> 
 <property name="basenames"> 
 <list> 
 <value>messages</value> //这个就是资源文件的名称
 <value>errorcodes</value> 
 </list> 
 </property> 
 </bean> 
 ... 
</beans> 
```

有了这些，我们就可以通过ApplicationContext直接访问相应Locale对应的信息，如下所示： 

```
ApplicationContext ctx = ...; 
String fileMenuName = ctx.getMessage("menu.file", new Object[]{"F"}, Locale.US); 
String editMenuName = ctx.getMessage("menu.file", null, Locale.US); 
assertEquals("File(F)", fileMenuName); 
assertEquals("Edit", editMenuName);
```

#### MessageSource实现类

Spring提供了三种MessageSource的实现，即StaticMessageSource、ResourceBundleMessageSource和ReloadableResourceBundleMessageSource。

-   org.springframework.context.support.StaticMessageSource。MessageSource接口的 简单实现，可以通过编程的方式添加信息条目，多用于测试，不应该用于正式的生产环境。 
-   org.springframework.context.support.ResourceBundleMessageSource。基于标准的 java.util.ResourceBundle而实现的MessageSource，对其父类AbstractMessageSource 的行为进行了扩展，提供对多个ResourceBundle的缓存以提高查询速度。同时，对于参数化 的信息和非参数化信息的处理进行了优化，并对用于参数化信息格式化的MessageFormat实 例也进行了缓存。它是最常用的、用于正式生产环境下的MessageSource实现。 

<!---->

-   org.springframework.context.support.ReloadableResourceBundleMessageSource 。 同样基于标准的java.util.ResourceBundle而构建的MessageSource实现类，但通过其 cacheSeconds属性可以指定时间段，以定期刷新并检查底层的properties资源文件是否有变更。 对于properties资源文件的加载方式也与ResourceBundleMessageSource有所不同，可以通过 ResourceLoader来加载信息资源文件。使用ReloadableResourceBundleMessageSource时， 应该避免将信息资源文件放到classpath中，因为这无助于ReloadableResourceBundleMessageSource定期加载文件变更。 

## 容器内部事件发布 

Spring的ApplicationContext容器提供的容器内事件发布功能，是通过提供一套基于Java SE标 准自定义事件类而实现的。为了更好地了解这组自定义事件类，我们可以先从Java SE的标准自定义事 件类实现的推荐流程说起。 

### Java SE的事件发布

Java SE提供了实现自定义事件发布（Custom Event publication）功能的基础类，即java.util.EventObject类和java.util.EventListener接口。所有的自定义事件类型可以通过扩展EventObject 来实现，而事件的监听器则扩展自EventListener。下面让我们看一下要实现一套自定义事件发布类的架构，应该如何来做。 

#### 1. 给出自定义事件类型（define your own event object）。

为了针对具体场景可以区分具体的事件 类型，我们需要给出自己的事件类型的定义，通常做法是扩展java.util.EventObject类来实现自定 义的事件类型。我们此次定义的自定义事件类型见如下代码：

```
public class MethodExecutionEvent extends EventObject { 
 private static final long serialVersionUID = -71960369269303337L; 
 private String methodName; 
 
 public MethodExecutionEvent(Object source) { 
 super(source); 
 } 
 public MethodExecutionEvent(Object source,String methodName) 
 { 
 super(source); 
 this.methodName = methodName; 
 } 
 public String getMethodName() { 
 return methodName; 
 } 
 public void setMethodName(String methodName) { 
 this.methodName = methodName; 
 } 
} 
```

我们想对方法的执行情况进行发布和监听，所以，就声明了一个MethodExecutionEvent类型， 它继承自EventObject，当该类型的事件发布之后，相应的监听器即可对该类型的事件进行处理。如果需要，自定义事件类可以根据情况提供更多信息，不用担心自定义事件类的“承受力”。 

#### 2. 实现针对自定义事件类的事件监听器接口（define custom event listener）和简单实现类

自定义的事件监听 器需要在合适的时机监听自定义的事件，如刚声明的MethodExecutionEvent，我们可以在方法开始执行的时候发布该事件，也可以在方法执行即将结束之际发布该事件。相应地，自定义的事件监听器 需 要提供方 法 对 这 两 种情况下接 收 到的事件进行处理。 下列代码给出了 针 对 MethodExecutionEvent的事件监听器接口定义 

```
public interface MethodExecutionEventListener extends EventListener { 
 /** 
 * 处理方法开始执行的时候发布的MethodExecutionEvent事件
 */ 
 void onMethodBegin(MethodExecutionEvent evt); 
 /** 
 * 处理方法执行将结束时候发布的MethodExecutionEvent事件
 */ 
 void onMethodEnd(MethodExecutionEvent evt); 
} 
```

事件监听器接口定义首先继承了java.util.EventListener，然后针对不同的事件发布时机提供 相应的处理方法定义，最主要的就是，这些处理方法所接受的参数就是MethodExecutionEvent类型 的事件。也就是说，我们的自定义事件监听器类只负责监听其对应的自定义事件并进行处理。以下是一个简单的实现类

```
public class SimpleMethodExecutionEventListener implements MethodExecutionEventListener { 
 public void onMethodBegin(MethodExecutionEvent evt) { 
	 String methodName = evt.getMethodName(); 
	 System.out.println("start to execute the method["+methodName+"]."); 
 } 
 public void onMethodEnd(MethodExecutionEvent evt) { 
 	String methodName = evt.getMethodName(); 
 	System.out.println("finished to execute the method["+methodName+"]."); 
 } 
} 
```

#### 3. 组合事件类和监听器，发布事件。 

有了自定义事件和自定义事件监听器，剩下的就是发布事件， 然后让相应的监听器监听并处理事件了。通常情况下，我们会有一个事件发布者（EventPublisher）， 它本身作为事件源，会在合适的时点，将相应事件发布给对应的事件监听器。下列代码给出了针 对MethodExecutionEvent的事件发布者类的定义。 

```
public class MethodExeuctionEventPublisher { 
 private List<MethodExecutionEventListener> listeners = new 	
ArrayList<MethodExecutionEventListener>(); 
 //要监听的方法
 public void methodToMonitor() { 
 	MethodExecutionEvent event2Publish = new MethodExecutionEvent(this,"methodToMonitor"); 
    //推送事件开始状态
 	publishEvent(MethodExecutionStatus.BEGIN,event2Publish); 
 	// 
    //这里执行实际的方法逻辑
 	// ......
     //推送事件结束状态
	 publishEvent(MethodExecutionStatus.END,event2Publish); 
 } 
 
 protected void publishEvent(MethodExecutionStatus status,
 MethodExecutionEvent methodExecutionEvent) { 
     //copy监听器列表
 	List<MethodExecutionEventListener> copyListeners = 
 new ArrayList<MethodExecutionEventListener>(listeners); 
 	for(MethodExecutionEventListener listener:copyListeners) { 
 		if(MethodExecutionStatus.BEGIN.equals(status)) 
 			listener.onMethodBegin(methodExecutionEvent); 
 		else 
 			listener.onMethodEnd(methodExecutionEvent); 
 	} 
 } 
 
 public void addMethodExecutionEventListener(MethodExecutionEventListener listener) 
 { 
 	this.listeners.add(listener); 
 } 
 public void removeListener(MethodExecutionEventListener listener) 
 { 
 	if(this.listeners.contains(listener)) 
 		this.listeners.remove(listener); 
 } 
 public void removeAllListeners() 
 { 
	 this.listeners.clear(); 
 } 
 public static void main(String[] args) { 
 MethodExeuctionEventPublisher eventPublisher = new MethodExeuctionEventPublisher(); 
 eventPublisher.addMethodExecutionEventListener(new SimpleMethodExecutionEventListener()); 
 eventPublisher.methodToMonitor(); 
 } 
    
}
```

我们的事件发布者关注的主要有两点：

**具体时点上自定义事件的发布**：方法methodToMonitor()是事件发布的源头，MethodExeuctionEventPublisher在该方法开始和即将结束的时候，分别针对这两个时点发布MethodExecutionEvent 事件。具体实现上，每个时点发布的事件会通过MethodExecutionEventListener的相应方法传给注 册的监听者并被处理掉。在实现中，需要注意到，为了避免事件处理期间事件监听器的注册或移除操 作影响处理过程，我们对事件发布时点的监听器列表进行了一个安全复制（safe-copy）。另外，事件 的发布是顺序执行，所以为了能够不影响处理性能，事件监听器的处理逻辑应该尽量简短。 

**自定义事件监听器的管理**：MethodExeuctionEventPublisher类提供了与事件监听器的注册和 移除相关的方法，这样，客户端可以根据情况决定是否需要注册或者移除某个事件监听器。这里容易 出现问题的情况是，如果没有提供remove事件监听器的方法，那么注册的监听器实例会一直被 MethodExeuctionEventPublisher引 用，即使已经 过 期 了 或 者 废 弃 不用了，也 依 然存在于 MethodExeuctionEventPublisher的监听器列表中。这会导致隐性的内存泄漏，在任何事件监听器 的处理上都可能出现这种问题。 

整个Java SE中标准的自定义事件实现就是这个样子，基本上涉及三个角色，即自定义的事件类型、 自定义的事件监听器和自定义的事件发布者，关系如图5-4所示。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a82fcdb7934418994926b5f223e6ea7~tplv-k3u1fbpfcp-zoom-1.image)

### Spring 的容器内事件发布类结构分析 

Spring 的 ApplicationContext 容器内部 允 许 以 org.springframework.context.ApplicationEvent的形式发布事件，容器内注册的org.springframework.context.ApplicationListener类型的bean定义会被ApplicationContext容器**自动识别**，它们负责监听容器内发布的所有 ApplicationEvent类型的事件。也就是说，一旦容器内发布ApplicationEvent及其子类型的事件， 注册到容器的ApplicationListener就会对这些事件进行处理。 不需要publisher指定。

#### ApplicationEvent 

Spring容器内自定义事件类型，继承自java.util.EventObject，它是一个抽象类，需要根据情 况提供相应子类以区分不同情况。默认情况下，Spring提供了三个实现。  

-   ContextClosedEvent：ApplicationContext容器在即将关闭的时候发布的事件类型。  
-   ContextRefreshedEvent：ApplicationContext容器在初始化或者刷新的时候发布的事件类型。 

<!---->

-   RequestHandledEvent：Web请求处理后发布的事件，其有一子类ServletRequestHandledEvent提供特定于Java EE的Servlet相关事件。 

#### ApplicationListener 

ApplicationContext容器内使用的自定义事件监听器接口定义，继承自java.util.EventListener。ApplicationContext容器在启动时，会自动识别并加载EventListener类型bean定义， 一旦容器内有事件发布，将通知这些注册到容器的EventListener

#### ApplicationEventMulticaster

虽然ApplicationContext 继承了ApplicationEventPublisher接口而担当了事件发布者的角色，但在具体实现上ApplicationContext是把活转让给ApplicationEventMulticaster。

该接口定义了具体事件监听器的注册管理以及事件发布的方法，ApplicationEventMulticaster有一抽象实现类——org.springframework.context.event.AbstractApplicationEventMulticaster，它实现了事件监听器的管理 功能。出于灵活性和扩展性考虑，事件的发布功能则委托给了其子类：org.springframework. context.event.SimpleApplicationEventMulticaster，它 是 Spring 提供的 AbstractApplicationEventMulticaster的一个子类实现，添加了事件发布功能的实现。不过，其默认使用了SyncTaskExecutor进行事件的发布。与我们给出的样例事件发布者实现一样，事件是同步顺序发布的。为了避 免 这种方式可能存在的性能问 题 ，我们可以为其提供其他类型的 TaskExecutor 实现类 （TaskExecutor的概念将在后面详细介绍）。 

因为ApplicationContext容器的事件发布功能全部委托给了ApplicationEventMulticaster 来做，所以，容器启动伊始，就会检查容器内是否存在名称为applicationEventMulticaster的 ApplicationEventMulticaster对象实例。有的话就使用提供的实现，没有则默认初始化一个 SimpleApplicationEventMulticaster作为将会使用的ApplicationEventMulticaster。这样，整 个Spring容器内事件发布功能实现结构图就有了，如图5-5所示。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2affb83f595841a491550aa7b6dbf2f3~tplv-k3u1fbpfcp-zoom-1.image)

### Spring 容器内事件发布的应用 

Spring的ApplicationContext容器内的事件发布机制，主要用于单一容器内的简单消息通知和处 理，并不适合分布式、多进程、多容器之间的事件通知。虽然可以通过Spring的Remoting支持，“曲 折一点”来实现较为复杂的需求，但是难免弊大于利，失大于得。其他消息机制处理较复杂场景或许 更合适。所以，我们应该在合适的地点、合适的需求分析的前提下，合理地使用Spring提供的 ApplicationContext容器内的事件发布机制。 

要让我们的业务类支持容器内的事件发布，需要它拥有ApplicationEventPublisher的事件发 布支持。所以，需要为其注入ApplicationEventPublisher实例。可以通过如下两种方式为我们的 业务对象注入ApplicationEventPublisher的依赖：

1.  使用ApplicationEventPublisherAware接口。在ApplicationContext类型的容器启动时， 会自动识别该类型的bean定义并将ApplicationContext容器本身作为ApplicationEventPublisher注入当前对象，而ApplicationContext容器本身就是一个ApplicationEventPublisher。 
1.  使用ApplicationContextAware接口。既然ApplicationContext本身就是一个ApplicationEventPublisher，那么通过ApplicationContextAware几乎达到第一种方式相同的效果。 

#### 1. MethodExecutionEvent的改装 

```
public class MethodExecutionEvent extends ApplicationEvent { 
 private static final long serialVersionUID = -71960369269303337L; 
 private String methodName; 
 private MethodExecutionStatus methodExecutionStatus; 
 
 public MethodExecutionEvent(Object source) { 
 super(source); 
 } 
 public MethodExecutionEvent(Object source,String methodName, 
 MethodExecutionStatus methodExecutionStatus) 
 { 
 super(source); 
 this.methodName = methodName; 
 this.methodExecutionStatus = methodExecutionStatus; 
 } 
 public String getMethodName() { 
 return methodName; 
 } 
 public void setMethodName(String methodName) { 
 this.methodName = methodName; 
 } 
 public MethodExecutionStatus getMethodExecutionStatus() { 
 return methodExecutionStatus; 
 }
 public void setMethodExecutionStatus(MethodExecutionStatus methodExecutionStatus) { 
 this.methodExecutionStatus = methodExecutionStatus; 
 } 
} 
```

#### 2. MethodExecutionEventListener 

我们的MethodExecutionEventListener不再是接口，而是具体的ApplicationListener实现 类。因为ApplicationListener已经取代了MethodExecutionEventListener原来的角色 

```
public class MethodExecutionEventListener implements ApplicationListener { 
 	public void onApplicationEvent(ApplicationEvent evt) { 
 		if(evt instanceof MethodExecutionEvent) 
 		{ 
 			// 执行处理逻辑
 		} 
 	} 
} 
```

#### 3. MethodExeuctionEventPublisher改造 

```
public class MethodExeuctionEventPublisher implements ApplicationEventPublisherAware { 
 private ApplicationEventPublisher eventPublisher; 
 
 public void methodToMonitor() 
 { 
 MethodExecutionEvent beginEvt = new 
 MethodExecutionEvent(this,"methodToMonitor",MethodExecutionStatus.BEGIN); 
 this.eventPublisher.publishEvent(beginEvt); 
 // 执行实际方法逻辑
 // ... 
 MethodExecutionEvent endEvt = new  MethodExecutionEvent(this,"methodToMonitor",MethodExecutionStatus.END); 
 this.eventPublisher.publishEvent(endEvt); 
 } 
    //该方法重写了ApplicationEventPublisherAware里的方法，在创建bean的时候
    //会自动调用该方法把ApplicationContext传进去
 public void setApplicationEventPublisher(ApplicationEventPublisher appCtx) { 
 this.eventPublisher = appCtx; 
 } 
} 
```

#### 4. 注册到ApplicationContext容器 

最后一步工作就是将MethodExeuctionEventPublisher和MethodExecutionEventListener注 册到ApplicationContext容器中。当MethodExeuctionEventPublisher的methodToMonitor方法被 调用时，事件即被发布。配置如下所示： 

```
<bean id="methodExecListener" class="...MethodExecutionEventListener"> 
</bean> 
<bean id="evtPublisher" class="...MethodExeuctionEventPublisher"> 
</bean>
```

```
        public static void main(String[] args) {
            ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
            MethodExeuctionEventPublisher pub = context.getBean("evtPublisher",MethodExeuctionEventPublisher.class);
            pub.methodToMonitor();
        }
```

若有收获，就点个赞吧

