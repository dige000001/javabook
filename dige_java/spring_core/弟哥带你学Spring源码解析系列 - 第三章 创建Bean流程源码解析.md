## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 核心类介绍

## DefaultListableBeanFactory

XmIBeanFactory继承自DefaultListableBeanFactory，而 DefaultListableBeanFactory是整个bean加载的核心部分，是Spring 注册及加载bean 的默认实现，而XmlBeanFactory 与DefaultListableBeanFactory不同的地方其实是在XmlBeanFactory中使用了自定义的XML读取器XmlBeanDefinitionReader，实现了个性化的 BeanDefinitionReader读取，DefaultListableBeanFactory继承了AbstractAutowireCapableBeanFactory并实现了ConfigurableListableBeanFactory以及BeanDefinitionRegistry 接口。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb0875373dd543efaf2f14600c6a55fe~tplv-k3u1fbpfcp-zoom-1.image)

各个类的作用：

```
AliasRegistry:定义对alias的简单增删改等操作。
SimpleAliasRegistry:主要使用map作为alias 的缓存，并对接口AliasRegistry进行实现。
SingletonBeanRegistry:定义对单例的注册及获取。
BeanFactory:定义获取bean 及bean的各种属性。
DefaultSingletonBeanRegistry:对接口SingletonBeanRegistry各函数的实现。
HierarchicalBeanFactory:继承BeanFactory，也就是在BeanFactory定义的功能的基础上增加了对parentFactory的支持。
BeanDefinitionRegistry:定义对BeanDefinition的各种增删改操作。
FactoryBeanRegistrySupport:在 DefaultSingletonBeanRegistry基础上增加了对FactoryBean的特殊处理功能。
ConfigurableBeanFactory:提供配置Factory的各种方法。
ListableBeanFactory:根据各种条件获取bean 的配置清单。
AbstractBeanFactory: 综合FactoryBeanRegistrySupport和ConfigurableBeanFactory 的功能。
AutowireCapableBeanFactory:提供创建bean、自动注入、初始化以及应用bean 的后处理器。
AbstractAutowireCapableBeanFactory:综合AbstractBeanFactory并对接口Autowire CapableBeanFactory进行实现。
ConfigurableListableBeanFactory: BeanFactory配置清单，指定忽略类型及接口等。
DefaultListableBeanFactory:综合上面所有功能，主要是对Bean注册后的处理。
```

## XmlBeanDefinitionReaderXML

配置文件的读取是Spring中重要的功能，因为Spring的大部分功能都是以配置作为切入点的,那么我们可以从XmlBeanDefinitionReader中梳理一下资源文件读取、解析及注册的大致脉络，首先我们看看各个类的功能。

```
ResourceLoader:定义资源加载器，主要应用于根据给定的资源文件地址返回对应的Resource.
BeanDefinitionReader:主要定义资源文件读取并转换为BeanDefinition的各个功能。
EnvironmentCapable:定义获取Environment方法。
DocumentLoader:定义从资源文件加载到转换为Document 的功能。
AbstractBeanDefinitionReader:对EnvironmentCapable、BeanDefinitionReader类定义的功能进行实现。
BeanDefinitionDocumentReader:定义读取Docuemnt并注册BeanDefinition功能
BeanDefinitionParserDelegate:定义解析Element的各种方法。
```

经过以上分析，我们可以梳理出整个XML配置文件读取的大致流程：

(1)通过继承自AbstractBeanDefinitionReader中的方法，来使用ResourLoader将资源文件路径转换为对应的 Resource文件。

(2）通过DocumentLoader对 Resource文件进行转换，将Resource文件转换为Document文件。

(3）通过实现接口 BeanDefinitionDocumentReader 的DefaultBeanDefinitionDocumentReader类对Document进行解析，并使用BeanDefinitionParserDelegate对Element进行解析。

# Step 1. 解析XML



## 1.加载配置文件，并得到对应的Document

```
BeanFactory bf = new XmlBeanFactory(new ClassPathResource ("beanFactoryTest.xml"));
```

很显然，这一步从路径中加载资源，在xmlBeanFactory的构造方法中，会调用内部Reader类的loadBeanDefinition(resource)加载资源。

在这个数据准备的过程中，首先对传入的resource参数做封装，目的是考虑到Resource可能存在编码要求的情况,其次,通过SAX读取XML文件的方式来准备InputSource对象,最后将准备的数据通过参数传入真正的核心处理部分doLoadBeanDefinitions(inputSource,encodedResource.getResource())。

而在doLoadBeanDefinitions方法里，完成了以下三个功能：

(1）getValidationModeForResource (resource) ; 获取对XML文件的验证模式。

(2）Document doc = this.documentLoader. loadDocument(...); 加载XML文件，并得到对应的Document。

(3）registerBeanDefinitions (doc, resource); 根据返回的 Document注册Bean信息。

## 2.解析及注册BeanDefinitions

当程序已经拥有XML文档文件的 Document实例对象时，就会被引入下面这个方法。

```
public int registerBeanDefinitions (Document doc，Resource resource) throws BeanDefinitionStoreException {
//使用DefaultBeanDefinitionDocumentReader实例化BeanDefinitionDocumentReader
	BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
//将环境变量设置其中
	documentReader.setEnvironment (this.getEnvironment()) ;
//在实例化BeanDefinitionReader时候会将 BeanDefinitionRegistry传入，默认使用继承自
//DefaultListableBeanFactory的子类
  //记录统计前BeanDefinition的加载个数
	int countBefore = getRegistry() .getBeanDefinitioncount() ;
  //加载及注册bean
	documentReader.registerBeanDefinitions(doc，createReaderContext(resource));
  //记录本次加载的BeanDefinition个数
return getRegistry ().getBeanDefinitioncount () - countBefore;
}
```

```
publie void registerBeanDefinitions (Documont doc，XmlRoaderContext readerContext){
	this.readerContext = readerContext;
	logger.debug ("Loading bean definitions");
    Element root = doc.getDocumentElement();
    doRegisterBeanDefinitions (root) ;
}
```

doRegisterBeanDefinitions是真正解析Bean的方法

```
protected void doRegisterBeanDefinitions(Element root){
	//处理profile属性
	string profileSpec = root.getAttribute(PROFILE_ATTRIBUTE) ;
    if(Stringutils.hasText (profileSpec))
	 {Assert.state(this.environment != null,"environment property must not be null");
 	string[]specifiedProfiles = stringutils.tokenizeTostringArray(profileSpec,
				BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
	if(!this.environment.acceptsProfiles(specifiedProfiles)) (
		 return;
	)
//专门处理解析
	BeanDefinitionParserDelegate parent = this.delegate;
	this.delegate = createHelper(readerContext,root,parent) ;
//解析前处理，留给子类实现
	preProcessXml (root) ;
	parseBeanDefinitions (root, this.delegate) ;
//解析后处理,留给子类实现
	postProcessxml (root) ;
    this.delegate = parent;
    }
```

#### PROFILE属性

我们注意到在注册Bean的最开始是对PROFILE_ATTRIBUTE属性的解析，可能对于我们来说，profile 属性并不是很常用。让我们先了解一下这个属性。分析profile前我们先了解下profile的用法，官方示例代码片段如下:

```
<beans profile="dev">
</beans>

<beans profile="production">
</beans>
```

```
集成到web环境中时，在 web.xml 中加入以下代码:
<context-param>
<param-name>Spring.profiles.active</param-name><param-value>dev</param-value>
</context-param>
```

有了这个特性我们就可以同时在配置文件中部署两套配置来适用于生产环境和开发环境，这样可以方便的进行切换开发、部署环境，最常用的就是更换不同的数据库。

了解了profile的使用再来分析代码会清晰得多，首先程序会获取 beans节点是否定义了profile属性，如果定义了则会需要到环境变量中去寻找，所以这里首先断言environment不可能为空，因为profile是可以同时指定多个的，需要程序对其拆分，并解析每个 profile是都符合环境变量中所定义的,不定义则不会浪费性能去解析。

### parseBeanDefinitions

处理了profile后就可以进行XML的读取了，跟踪代码进入 parseBeanDefinitions(root,this.delegate)。

```
protected void parseBeanDefinitions(Element root，BeanDefinitionParserDelegate delegate) {
//对beans的处理
	if (delegate.isDefaultNamespace (root)) {
		NodeList nl = root.getChildNodes ( ) ;
		for (int i -0; i < nl.getLength () ; i++) {
			  Node node = nl.item (i) ;
			  if (node instanceof Element) {
				Element ele = (Element) node;
				if (delegate.isDefaultNamespace (ele)){
		          //对bean的处理
					parseDefaultElement (ele, delegate) ;
                }
				else {
				  //对bean的处理
					delegate.parsecustomElement (ele);
				}
                }
              }
        }
    else {delegate.parseCustomElement (root);}
}
```

上面的代码看起来逻辑还是蛮清晰的，因为在Spring 的XML 配置里面有两大类Bean声明，一个是默认的，如: <bean id="test" class="test.TestBean" /> 另一类就是自定义的，如; <tx : annotation-driven/>

而两种方式的读取及解析差别是非常大的，如果采用Spring默认的配置，Spring当然知道该怎么做，但是如果是自定义的，那么就需要用户实现一些接口及配置了。对于根节点或者子节点如果是默认命名空间的话则采用parseDefaultElement方法进行解析，否则使用delegate.parseCustomElement方法对自定义命名空间进行解析。而判断是否默认命名空间还是自定义命名空间的办法其实是使用node.getNamespaceURI()获取命名空间，并与Spring中固定的命名空间http://www.Springframework.org/schema/beans进行比对。如果一致则认为是默认，否则就认为是自定义。而对于默认标签解析在下一章中进行讨论。

## 3.默认标签的解析

默认标签的解析是在parseDefaultElement 函数中进行的

```
private void parseDefaultElement(Element ele，BeanbefinitionParserDelegate delegate){
 //对import标签的处理
	if (delegate.nodeNameEquals(ele,IMPORT_ELEMENT)){
		importBeanDefinitionResource ( ele) ;}
    
//对alias标签的处理
else if (delegate.nodeNameEquals(ele,ALIAS_ELEMENT)){
processAliasRegistration (ele) ;}
    
//对bean标签的处理
else if (delegate.nodeNameEquals(ele，BEAN_ELEMENT)){
processBeanDefinition (ele, delegate) ;}
    
//对beans标签的处理
else if (delegate.nodeNameEquals(ele，NESTED_BEANS_ELEMENT)){
doRegisterBeanDefinitions (ele) ;}
```

函数中的功能逻辑一目了然，分别对4种不同标签( import、alias、bean和 beans)做了不同的处理。

**这里只讨论<bean>标签**

```
protected void processBeanDefinition(Element ele，BeanDefinitionParserDelegate delegate) {
	BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionBlement(ele);
    if(bdHolder != null){
		bdHolder = delegate.decorateBeanDefinitionIfRequired(ele，bdHolder) ;
        try {
		 	BeanDefinitionReaderUtils.registerBeanDefinition (bdHolder,getReaderContext (),
			       getRegistry ());
		 }
		catch (BeanDefinitionstoreException ex) {
		   getReaderContext ().error("Failed to register bean definition with name '" +
				bdHolder.getBeanName () + "'", ele, ex);
               }
    getReaderContext().fireConponentPegistered(new BeanComponentDefinition (bdlolder));
    }
}
```

逻辑如下：

(1）首先委托BeanDefinitionDelegate类的 parseBeanDefinitionElement 方法进行元素解析,返回 BeanDefinitionHolder类型的实例bdHolder，经过这个方法后，bdHolder实例已经包含我们配置文件中配置的各种属性了，例如 class、name、id、alias 之类的属性。

(2）当返回的 bdHolder 不为空的情况下若存在默认标签的子节点下再有自定义属性，还需要再次对自定义标签进行解析。

(3）解析完成后，需要对解析后的 bdHolder进行注册，同样，注册操作委托给了BeanDefinitionReaderUtils 的registerBeanDefinition方法。

(4）最后发出响应事件，通知想关的监听器，这个bean已经加载完成了。

### 解析BeanDefinition

下面我们就针对各个操作做具体分析。首先我们从元素解析及信息提取开始，也就是

BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele)，进入 BeanDefinitionDelegate类的 parseBeanDefinitionElement方法。

```
public BeanbefinitionHolder parseBeanbefinitionElement(Element ele) {
return parseBeanDefinitionElement(ele, null) ;
}

public BeanDefinitionHolder parseBeanDefinitionElement(Element ele，BeanDefinitioncontainingBean) {
//解析id属性
	string id = ele.getAttribute(ID_ATTRIBUTE);
//解析name属性
	string nameAttr = ele.getAttribute (NAME_ATTRIBUTE);
//分割name属性
	List<string> aliases = new ArrayList<string>();
    if (stringutils.hasLength (nameAttr)){
			string[] nameArr = stringUtils.tokenizeToStringArray (nameAttr，MULTI_
VALUE_ATTRIBUTE_DELIMITERS);
			aliases.addAll(Arrays.asList (nameArr));
    }
	string beanName = id;
	if (!StringUtils.hasText(beanName) &&!aliases.isEmpty()) {
			beanName = aliases.remove(0);
			if (logger.isDebugEnabled ()){
				logger.debug ( "No XML 'id' specified - using '" + beanName +
 " as bean name and " + aliases + " as aliases");
            }
    }
	if (containingBean == null){
			checkNameUniqueness(beanName, aliases, ele) ;
		}
 //解析其他属性
	AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele，beanName,
containingBean);
    /* 
    下面就是生成beanName等细枝末节
    这里不贴代码了
	*/
    
   	string[] aliasesArray = stringUtils.toStringArray(aliases);
//将以上获取到的所有信息封装到BeanDefinitionHolder中
	return new BeanDefinitionHolder(beanDefinition，beanName，aliasesArray);

    
```

主要实现以下几个功能：

(1）提取<bean>标签元素中的id以及name属性。

(2）进一步解析其他所有属性并统一封装至GenericBeanDefinition类型的实例中。

(3）如果检测到bean没有指定beanName，那么使用默认规则为此Bean 生成beanName。

(4）将获取到的信息封装到BeanDefinitionHolder的实例中。

在（2）中，主要执行以下步骤：

1.  创建用于属性承载的GenericBeanDefinition实例
1.  解析元素的各种属性

<!---->

3.  解析子标签<meta>
3.  解析子标签<lookup-method>

<!---->

5.  解析子标签<replace-method>
5.  解析构造标签<constructor-arg>

<!---->

7.  解析子标签<property>
7.  解析子元素<qualifier>

### 注册解析的BeanDefinition

对于配置文件，解析也解析完了，装饰也装饰完了，对于得到的 beanDinition已经可以满足后续的使用要求了，唯一还剩下的工作就是注册了，也就是 processBeanDefinition函数中的BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext0.getRegistry())代码的解析了。

```
public static void registerBeanDefinition (
	BeanDefinitionHolder definitionHolder，BeanDefinitionRegistry registry)throws 
      BeanDefinitionStoreException{
//使用beanName做唯一标识注册
string beanName = definitionHolder.getBeanName ( ) ;
registry.registerBeanDefinition(beanName，definitionHolder. getBeanDefinition());
//注册所有的别名
String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
		for (string aliase : aliases){
				registry.registerAlias (beanName, aliase) ;
        }
    }
}
```

从上面的代码可以看出，解析的 beanDefinition都会被注册到 BeanDefinitionRegistry类型的实例registry 中，而对于beanDefinition的注册分成了两部分：通过beanName的注册以及通过别名的注册。

#### 通过beanName的注册：

在对于bean的注册处理方式上，主要进行了几个步骤。

(1)对AbstractBeanDefinition的校验。在解析XML文件的时候我们提过校验，但是此校验非彼校验，之前的校验时针对于XML格式的校验，而此时的校验时针是对于AbstractBeanDefinition的methodOverrides属性的。

(2）对 beanName已经注册的情况的处理。如果设置了不允许bean的覆盖，则需要抛出异常，否则直接覆盖。

(3）加入map缓存。

(4）清除解析之前留下的对应beanName的缓存。

#### 通过别名的注册

( 1 ) alias 与 beanName相同情况处理。若alias 与 beanName名称相同则不需要处理并删除掉原有alias。

( 2 ) alias覆盖处理。若aliasName已经使用并已经指向了另一beanName则需要用户的设置进行处理。

( 3 ) alias循环检查。当A->B存在时，若再次出现A->C->B时候则会抛出异常。

(4）注册alias。

## <alias>、<import>、<beans>标签的解析和注册 

暂时略过

## 自定义标签的解析和注册

暂时略过



# Step 2. bean的加载

bean的加载是在调用了getbean()后进行的

```
public object getBean (String name) throws BeansException {
		return doGetBean (name,null,null,false ) ;
}

protected <T> T doGetBean (
			final string name, final class<T> requiredType，final object [] args,boolean
typeCheckonly) throws BeansException {
//提取对应的beanName
			final String beanName = transformedBeanName (name ) ;
            Object bean;
/*
*检查缓存中或者实例工厂中是否有对应的实例
为什么首先会使用这段代码呢,
因为在创建单例bean 的时候会存在依赖注人的情况，而在创建依赖的时候为了避免循环依赖，spring创建bean 的原则是不等bean创建完成就会将创建bean的ObjectFactory提早曝光
也就是将ObjectFactory加入到缓存中，一旦下个bean创建时候需要依赖上个bean则直接使用ObjectFactory
*/
//直接尝试从缓存获取或者singletonFactories中的objectFactory中获取
	Object sharedInstance = getsingleton (beanName);
	if (sharedInstance != null && args == null){
		if (logger.isDebugEnabled()) {
			if (isSingletonCurrentlyInCreation (beanName)) {
				logger.debug ("Returning eagerly cached instance of singleton bean
'"+beanName +
"' that is not fully initialized yet - a consequence of a circular reference") ;
			}
            else {
			logger.debug("Returning cached instance of singleton bean '" +beanName + "'");
			 }
		 }
//返回对应的实例，有时候存在诸如BeanFactory 的情况并不是直接返回实例本身而是返回指定方法返回的实例
		bean = getobjectForBeanInstance(sharedInstance，name，beanName，null);
	} else {
//只有在单例情况才会尝试解决循环依赖，原型模式情优下，如果仔在
//A中有B的属性,B中有A的属性，那么当依赖注人的时候，就会产生当A还未创建完的时候因为
//对于B的创建再次返回创建A，造成循环依赖，也就是下面的情况
//isPrototypeCurrentlyInCreation (beanName)为true
            if (isPrototypeCurrentlyInCreation(beanName)) {
                throw new BeanCurrentlyInCreationException(beanName);
            }
            //如果 beanDefinitionMap中也就是在所有已经加载的类中不包括 beanName 则尝试从				//parentBeanFactory中检测
		 	if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
                String nameToLookup = originalBeanName(name);
			//递归到 BeanFactory中寻找
            	if (args != nul1) {
					return (T) parentBeanFactory.getBean (nameToLookup,args);
                  }
				else {
					return parentBeanFactory.getBean (nameToLookup,requiredType) ;
					}
                }
				//如果不是仅仅做类型检查则是创建bean，这里要进行记录
  				if (!typeCheckOnly) {
                	markBeanAsCreated(beanName);
            	}
				//将存储XML配置文件的GernericBeanDefinition转换为RootBeanbefinition，如果指
				//定BeanName是子Bean的话同时会合并父类的相关属性
				final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
				checkMergedBeanDefinition (mbd,beanName,args) ;
				String []dependsOn = mbd.getDependsOn ( ) ;
				//若存在依赖则需要递归实例化依赖的bean
				if (dependsOn != null) {
					 for (String dependsOnBean : dependsOn) {
						  getBean (dependsOnBean);
				//缓存依赖调用
							registerDependentBean (dependsonBean,beanName);
                         }
                  }
                 //实例化依赖的bean后便可以实例化 mbd本身了
                 //singleton模式的创建
				 if(mbd.issingleton()) {
					sharedInstance = getsingleton (beanName，new objectFactory<Object>() {
						public Object getObject () throws BeansException {
							try {
								 return createBean (beanName,mbd, args);
                               }
							catch (BeansException ex){
								destroySingleton (beanName) ;
                                throw ex;
							}
                         }
                    });
                    bean = getobjectForBeanInstance(sharedInstance，name，beanName，mbd) ;
                   }
                   else if (mbd.isPrototype ()) {
						//prototype模式的创建(new)
						Object prototypeInstance = null;
                        try {
							beforePrototypeCreation (beanName) ;
							prototypeInstance = createEean (beanName,mbd,args) ;
                         }
						finally {
									afterPrototypeCreation (beanName);
                         }
					 bean = getObjectForBeanInstance(prototypeInstance name,beanName,mbd);
                    }
                    else {
						//指定的scope 上实例化bean
						String scopeName = mbd.getscope();
						final scope scope = this.scopes.get (scopeName) ;
                        if (scope == null) {
						throw new IllegalstateException ("No scope registered for scope '"+scopeName +"'") ;
						}
  						try{
							Object scopedInstance = scope.get (beanName，new ObjectFactory<Object>() {
						public object getObject () throws BeansException {
							beforePrototypeCreation (beanName);
							try{
									return createBean (beanName,mbd,args) ;
                             }
							finally {
								afterPrototypeCreation (beanName) ;
                             }
                         }
					});
					 bean = getObjectForBeanInstance(scopedInstance，name,beanName，mbd);
                        }
                        catch (IllegalstateException ex) {
							throw new BeanCreationException (beanName,
									"scope '" + scopeName + "' is not active for the current thread; " + "consider defining a scoped proxy for this bean if you intend to refer to it from a singleton" , ex);
						}
					}
		}//对应32行的else
 	   //检查需要的类型是否符合bean的实际类型
       if (requiredType != null && bean != null 
        && !requiredType.isAssignableFrom(bean.getClass())) 
        {
			try {
				return getTypeConverter ( ) .convertIfNecessary (bean，requiredType);
             }
			catch (TypeMismatchException ex) {
				if (logger.isDebugEnabled()) {
					logger.debug ( "Failed to convert bean '" + name + "' to required type [" + ClassUtils.getQualifiedName (requiredType)+ "]", ex);
                   }
				throw new BeanNotofRequiredTypeException(name，requiredType，bean.
getclass());	
 			 }
		}
		return(T) bean;
      }
```

讲解：

**(1)转换对应beanName**。

或许很多人不理解转换对应beanName是什么意思，传入的参数name不就是 beanName吗?其实不是，这里传人的参数可能是别名，也可能是FactoryBean，所以需要进行一系列的解析，这些解析内容包括如下内容：

-   去除FactoryBean的修饰符，也就是如果name="&aa"，那么会首先去除&而使name="aa"
-   取指定alias 所表示的最终 beanName，例如别名A指向名称为B的 bean则返回B；若别名A指向别名B，别名B又指向名称为C的bean则返回C.

**(2)尝试从缓存中加载单例。**

单例在Spring 的同一个容器内只会被创建一次，后续再获取 bean,就直接从单例缓存中获取了。当然这里也只是尝试加载，首先尝试从缓存中加载，如果加载不成功则再次尝试从singletonFactories中加载。因为在创建单例bean的时候会存在依赖注入的情况，而在创建依赖的时候为了避免循环依赖,在Spring中创建bean的原则是不等bean创建完成就会将创建bean的ObjectFactory提早曝光加入到缓存中，一旦下一个bean创建时候需要依赖上一个bean则直接使用ObjectFactory (后面章节会对循环依赖重点讲解)。

**(3) bean的实例化。**

如果从缓存中得到了bean的原始状态，则需要对bean进行实例化。这里有必要强调一下，缓存中记录的只是最原始的 bean 状态，并不一定是我们最终想要的 bean。举个例子，假如我们需要对工厂bean进行处理，那么这里得到的其实是工厂bean的初始状态，但是我们真正需要的是工厂bean 中定义的 factory-method方法中返回的bean，而 getObjectForBeanInstance就是完成这个工作的，后续会详细讲解。

**(4）原型模式的依赖检查。**

只有在单例情况下才会尝试解决循环依赖，如果存在A中有B的属性,B中有A的属性，那么当依赖注入的时候，就会产生当A还未创建完的时候因为对于B的创建再次返回创建A,造成循环依赖，也就是情况:isPrototypeCurrentlyInCreation(beanName)判断true。

**(5）检测parentBeanFactory。**

从代码上看，如果缓存没有数据的话直接转到父类工厂上去加载了，!containsBeanDefinition(beanName)就比较重要，它是在检测如果当前加载的XML 配置文件中不包含beanName所对应的配置，就只能到parentBeanFactory去尝试下了，然后再去递归的调用getBean方法。

**(6）将存储XML配置文件的GernericBeanDefinition转换为RootBeanDefinition。**

因为从XML 配置文件中读取到的Bean信息是存储在GernericBeanDefinition中的，但是所有的 Bean后续处理都是针对于RootBeanDefinition 的，所以这里需要进行一个转换，转换的同时如果父类bean不为空的话，则会一并合并父类的属性。

**( 7)寻找依赖。**

因为 bean 的初始化过程中很可能会用到某些属性，而某些属性很可能是动态配置的，并且配置成依赖于其他的 bean，那么这个时候就有必要先加载依赖的bean，所以，在Spring 的加载顺寻中，在初始化某一个bean的时候首先会初始化这个bean所对应的依赖。

**( 8）针对不同的scope进行bean的创建。**

我们都知道，在Spring 中存在着不同的 scope，其中默认的是singleton，但是还有些其他的配置诸如prototype、request之类的。在这个步骤中，Spring 会根据不同的配置进行不同的初始化策略。

**( 9）类型转换。**

程序到这里返回bean后已经基本结束了,通常对该方法的调用参数requiredType是为空的,但是可能会存在这样的情况，返回的 bean其实是个String，但是requiredType却传入Integer类型,那么这时候本步骤就会起作用了,它的功能是将返回的 bean转换为requiredType所指定的类型。当然，String转换为Integer是最简单的一种转换，在Spring 中提供了各种各样的转换器，用户也可以自己扩展转换器来满足需求。

## 从缓存中获取bean

Object sharedInstance = getsingleton (beanName);

Spring使用三级缓存存放bean的不同map，这是为了解决循环依赖而设计的:

-   **singletonObjects**：一级缓存。用于保存BeanName和创建bean实例之间的关系，即这保存的是最终版本的bean
-   **earlySingletonObjects**：二级缓存。也是保存 BeanName和创建bean实例之间的关系，与singletonObjects 的不同之处在于，当一个单例 bean 被放到这里面后，那么当 bean还在创建过程中，就可以通过getBean方法获取到了。其存放的是提前暴露代理的版本的bean，这样，若有多个bean与当前bean循环依赖，从二级缓存获取可以保证获取的当前bean都是一样的

<!---->

-   **singletonFactories**：三级缓存。用于保存BeanName和创建bean的工厂之间的关系，其目的是用来检测循环引用。每次调用该工厂返回的提前暴露的代理bean的内存地址都不一样，故在第一次调用时，需要将获得的代理bean放到二级缓存
-   **registeredSingletons：** 记录所有已注册的bean



## 从头创建一个bean

上面讲的是从缓存中获取bean，而如果缓存中不存在已经加载的单例bean，就需要我们从头加载一个bean，Sprng使用getSingleton的重载方法实现bean的加载过程

public Object getSingleton (String beanName，ObjectFactory singletonFactory)

```
public object getsingleton(String beanName，ObjectFactory singletonFactory){
	Assert.notNull(beanName， "'beanName ' must not be null");
	//全局变量需要同步
	synchronized (this.singletonobjects) {
	//首先检查对应的bean是否已经加载过，因为singleton模式其实就是复用已创建的bean
	Object singletonObject = this.singletonObjects.get (beanName);
    //如果为空才可以进行singleto的bean的初始化
	if (singletonobject == null) {
		if(this.singletonsCurrentlyInDestruction){
			throw new BeanCreationNotAllowedException (beanName,
			"Singleton bean creation not allowed while the singletons of this factory are 				in destruction" +
			"(Do not request a bean from a BeanFactory in a destroy method implementation ! 			) ");
        }
		if (logger.isDebugEnabled ()){
			logger.debug ("creating sharedinstance of singleton bean ' " + beanName + "'");
         }
		beforeSingletonCreation(beanName) ;
		boolean recordsuppressedExceptions = (this.suppressedExceptions m= null);
        if (recordsuppressedExceptions) {
				this.suppressedExceptions = new LinkedHashSet<Exception>();
        }
		try {
			//实例化，属性注入，初始化bean
			singletonObject = singletonFactory.getObject();
		}
		catch (BeancreationException ex) {
			if(recordsuppressedExceptions) {
				for (Exception suppressedException : this.suppressedExceptions){
						ex.addRelatedcause (suppressedException) ;
                }
            }
			throw ex;
        }
		finally {
			if (recordsuppressedExceptions) {
                this.suppressedExceptions =null;
            }
			afterSingletonCreation (beanName) ;
        }
		//加入缓存
		addSingleton (beanName, singletonobject);
    }
	return (singletonObject != NULI_OBJECT ? singletonobject : null);
    }
  }
```

主要做了几件事：

1.  检查缓存中是否已经加载过该bean
1.  若没有加载，则记录BeanName的正在加载状态：beforeSingletonCreation(beanName) ;，这样就可以对循环依赖进行检测

<!---->

3.  实例化bean：singletonobject = singletonFactory.getObject();
3.  调用加载单例后的处理方法 afterSingletonCreation(beanName) ;，移除BeanName的正在加载状态

<!---->

5.  将加载的单例记录至缓存并删除加载过程所记录的各种辅助状态：也就是将bean放入一级缓存和registeredSingletons，并删除在二级缓存和三级缓存的内容
5.  返回该单例对象



singletonFactory.getObject();中调用了createBean()方法，从这里开始创建bean

### createBean()

在这个方法里，主要调用了三个方法：

-   **prepareMethodOverrides()**

在Spring配置中存在 lookup-method和 replace-method ，而这两个配置的加载其实就是将配置统一存放在BeanDefinition中的methodOverrides属性里,而这个函数的操作其实也就是针对于这两个配置的，在bean实例化的时候如果检测到存在methodOverrides属性，会动态地为当前bean 生成代理并使用对应的拦截器为bean做增强处理，相关逻辑实现在 bean的实例化部分详细介绍。

-   **resolveBeforeInstantiation()**

在这个方法中，如果创建了代理或者说重写了InstantiationAwareBeanPostProcessor 的postProcessBeforeInstantiation方法，则调用applyBeanPostProcessorsBeforeinstantiation和applyBeanPostProcessorsAfterInitialization生成bean，再直接返回就可以了，否则需要进行常规 bean的创建，即下面的doCreateBean()。

****

-   **doCreateBean()**

[Spring源码最难问题《当Spring AOP遇上循环依赖》_bugpool的博客-CSDN博客](https://blog.csdn.net/chaitoudaren/article/details/105060882)

主要做了以下几个工作：

1.  如果是单例则需要首先清除缓存。
1.  实例化bean，将BeanDefinition转换为Bean Wrapper。

转换是一个复杂的过程，但是我们可以尝试概括大致的功能，如下所示。

-   -   如果存在工厂方法则使用工厂方法进行初始化。
    -   一个类有多个构造函数，每个构造函数都有不同的参数，所以需要根据参数锁定构造函数并进行初始化。

<!---->

-   -   如果既不存在工厂方法也不存在带有参数的构造函数，则使用默认的构造函数进行bean的实例化。

3.  MergedBeanDefinitionPostProcessor的应用。bean合并后的处理，Autowired注解正是通过此方法实现诸如类型的预解析。
3.  依赖处理

<!---->

5.  **属性填充。** 将所有属性填充至bean的实例中。**这部分可能是重点，看书吧，实在不想写了**
5.  循环依赖检查。

在Sping中解决循环依赖只对单例有效，而对于prototype的 bean，Spring没有好的解决办法，唯一要做的就

是抛出异常。在这个步骤里面会检测已经加载的bean是否已经出现了依赖循环，并判断是否需要抛出异常。

7.  注册DisposableBean。

如果配置了destroy-method，这里需要注册以便于在销毁时候调用。

8.  完成创建并返回。

## Spring解决循环依赖



**Spring只能解决** **单例** **的** **setter方法注入** **的循环依赖**

实例化A后，将A的对象工厂方法ObjectFactory放入三级缓存，以供循环依赖\
开始注入b属性，发现对象B没有创建(一级缓存二级缓存都没有，三级缓存也没有)，去实例化B\
实例化B后，将B的对象工厂方法ObjectFactory放入三级缓存\
开始注入a属性,发现A没有创建，但是有一个ObjectFactory，调用getEarlyBeanReference，将返回的A放入二级缓存，将三级缓存中的A ref删除，并将A注入B的a属性，完成B的初始化，将B ref从三级缓存删除，将B放入一级缓存。返回A的初始化过程\
返回后，A再次检查，发现一级缓存中有B，于是将B注入A的a属性，将A从二级缓存删除，放入一级缓存

于是两个bean都创建完毕了，并且相互依赖


如果一个对象需要被代理的话，在整个容器中，会存在两个当前对象的版本\
第一个：原始对象（raw），直接通过反射创建出来的对象\
第二个：通过jdk动态代理或者cglib创建出来的代理对象




在bean实例化过程，doCreateBean()生成的是原始版本，而代理是发生在getEarlyBeanReference里的，而这个函数最终返回的，就是代理对象（这个过程发生在上面标红字的地方），如果该类没被代理，则返回的就是原始版本，空间地址都一样，但如果代理了，返回的就是全新的空间



**问：**

```
假设有两个类A和B，相互依赖，且A和B都被代理
那么两个类在创建过程中有以下流程：

1. 实例化A —> 将A的对象工厂放入三级缓存 —> 为A注入属性 —>发现B未实例化
2. 实例化B —> 将B的对象工厂放入三级缓存 —> 为B注入属性 —>从三级缓存中获取A的代理对象（将A的代理对	 象称为A1）并将A1注入B—>删除三级缓存中A的对象工厂，将A1放入二级缓存—>将三级缓存中B的对象工厂删	 除，将B放入一级缓存
3. A从一级缓存中将B注入自身，放入一级缓存。

现在我有个问题：
注入B的是A的代理对象A1，而注入A的是最终初始化后的B，也就是
A1—>B，B—>A，这似乎不符合循环依赖
```

**答：**

```
A1至始至终都存放在二级缓存，存在提前曝光的情况下，A完成属性注入及初始化后会赋给一个叫exposedObject的变量，而exposedObject在之后又会指向从二级缓存取出的A1，那么A1的属性注入和初始化又是在哪里完成的呢？：
JDK动态代理时，会将目标对象target保存在最后生成的代理$proxy中，当调用$proxy方法时会回调h.invoke，而h.invoke又会回调目标对象target的原始方法。因此，其实在Spring AOP动态代理时，原始bean已经被保存在提前曝光代理中了。而后原始Bean继续完成属性填充和初始化操作。因为AOP代理$proxy中保存着traget也就是是原始bean的引用，因此后续原始bean的完善，也就相当于Spring AOP中的target的完善，这样就保证了Spring AOP的属性填充与初始化了！
```

[Spring源码最难问题《当Spring AOP遇上循环依赖》_bugpool的博客-CSDN博客](https://blog.csdn.net/chaitoudaren/article/details/105060882)

**只用一级缓存无法满足多线程要求，只能解决单线程的循环依赖！不能处理并发和AOP**\
**只有二级缓存无法完成AOP需求**

**之所以用三级缓存：** 我觉得有用空间换时间的原理\
如果用二级缓存，意味着所有的bean都要走从对象工厂获取代理对象（或原对象）并放入二级缓存的过程，但不是每个bean都有循环依赖，不是每个bean都会被提前引用，所以用一个三级缓存来解决这部分特殊bean。\
而且只用二级缓存的话，bean的生命周期也要变一下了，没必要将代理的步骤放在BeanPostProcessor阶段，而是放在实例化之后的阶段，而Spring不选择这么做，肯定有他的原因

## 初始化bean

Spring 中程序已经执行过bean的实例化并且进行了属性的填充，而就在这时将会调用用户设定的初始化方法。

```
protected Object initializeBean (final String beanName，final Object bean，RootBeanDefinition mbd){
	if (system.getSecurityManager () != null) {
		Accesscontroller.doPrivileged(new PrivilegedAction<Object>() {
			public object run () {
				invokeAwareMethods (beanName, bean) ;
                return null;
            }
        },getAccessControlContext ());
    }
	else {
//对特殊的bean处理:Aware、BeanclassLoaderAware、BeanFactoryAware
        invokeAwareMethods(beanName, bean);
        Object wrappedBean = bean;
    }
	if (mbd == null || !mbd.issynthetic() ) {
		//应用后处理器
		wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean,beanName);
	}
	try {
		//激活用户自定义的init方法
		invokeInitMethods (beanName,wrappedBean,mbd) ;
    }
	catch (Throwable ex) {
		throw new BeancreationException ((mbd != null ? mbd.getResourceDescription():null), 										 beanName, "Invocation of init method failed",ex);
    }
	if(mbd == null || !mbd.isSynthetic()) {
	//后处理器应用
		wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean,beanNane);
    }
	return wrappedBean;
}
```

可以看到，主要做几个功能：

-   激活Aware接口
-   调用BeanPostProcessor的两个方法

<!---->

-   激活init-method方法


## 检测是否为FactoryBean

在getBean方法中，getObjectForBeanInstance是个高频率使用的方法，无论是从缓存中获得 bean还是根据不同的scope策略加载bean。总之，我们得到bean 的实例后要做的第一步就是调用这个方法，其实就是**用于检测当前bean是否是 FactoryBean类型的 bean，如果是，那么需要调用该bean对应的FactoryBean实例中的getObject()作为返回值**。无论是从缓存中获取到的bean还是通过不同的scope策略加载的bean并不一定是我们最终想要的 bean。举个例子，假如我们需要对工厂bean进行处理，那么这里得到的其实是工厂 bean的初始状态，但是我们真正需要的是工厂 bean中定义的factory-method方法中返回的 bean，而getObjectForBeanInstance方法就是完成这个工作的。

```
protected object getObjectForBeanInstance (
Object beanInstance，String name，String beanName，RootBeanDefinition mbd){
//如果指定的name是工厂相关(以&为前缀)且beanInstance又不是FactoryBean类型则验证不通过
    if (BeanFactoryUtils.isFactoryDereference (name) && ! (beanInstance instanceof
FactoryBean) ){
		throw new BeanIsNotAFactoryException (transformedBeanName (name), beanInstance.
getClass ());
    }
//现在我们有了个bean的实例，这个实例可能会是正常的bean或者是FactoryBean
//如果是FactoryBean我们使用它创建实例，但是如果用户想要直接获取工厂实例而不是工厂的
//getobject方法对应的实例那么传入的name应该加人前缀&
//若bean是普通类型，直接返回
	if (!(beanInstance instanceof FactoryBean)|| BeanFactoryUtils.IsFactoryDereference(name)) {
			return beanlnstance;
    }
	//加载FactoryBean
	Object object = null;
    if(mbd == null){
	//尝试从缓存中加载bean
	object = getCachedobjectForFactoryBean (beanName) ;
    }
	if (object == null) {
		//到这里已经明确知道beanInstance一定是FactoryBean类型
		FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
		// containsBeanDefinition检测beanDefinitionMap中也就是在所有已经加载的类中检测
		//是否定义beanName
		if (mbd == null && containsBeanDefinition(beanName)){
			//将存储XML配置文件的GernericBeanDefinition 转换为RootBeanDefinition,
			//如果指定BeanName是子 Bean的话同时会合并父类的相关属性
			mbd = getMergedLocalBeanDefinition (beanName) ;
		}
		//是否是用户定义的而不是应用程序本身定义的
		boolean synthetic = (mbd != null & & mbd.issynthetic() ) ;
		object = getObjectFromFactoryBean (factory，beanName，!synthetic);
    }
	return object;
}
```

getObjectForBeanInstance 中的所做的工作。

1.  对FactoryBean正确性的验证。
1.  对非FactoryBean不做任何处理。

<!---->

3.  对 bean进行转换。
3.  将从Factory 中解析bean的工作委托给getObjectFromFactoryBeano

可以看到，核心代码在34行的getObjectFromFactoryBean

```
protected Object getObjectFromFactoryBean(FactoryBean factory，string beanName，booleanshouldPostProcess) {
	//如果是单例模式
	if (factory.isSingleton () && containsSingleton(beanName)){
		synchronized (getSingletonMutex()) {
			Object object = this.factoryBeanObjectCache.get(beanName);
            if (object == null) {
				object=doGetObjectFromFactoryBean(factory, beanName,shouldPostProcess);
				this.factoryBeanObjectCache.put(beanName，(object !=  null? object:
NULL_OBJECT));
            }
	return (object != NULL_OBJECT ? object : null);
        }
    }
	else {
		return doGetObjectFromFactoryBean(factory,beanName，shouldPostProcess);
    }
```

**在这个方法里只做了一件事情,就是返回的 bean如果是单例的，那就必须要保证全局唯一，同时，也因为是单例的，所以不必重复创建，可以使用缓存来提高性能，也就是说已经加载过就要记录下来以便于下次复用，否则的话就直接获取了。** 获取对象是在doGetObjectFromFactoryBean中进行的

代码略

主要做了两件事：

1.  object =factory.getObject(); 即从工厂中返回真正需要的对象
1.  object = postProcessObjectFromFactoryBean(object,beanName) ; 调用ObjectFactory的后处理器

**Fin.**

