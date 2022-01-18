## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂


Application的拓展功能全在refresh()函数里

```
public void refresh ( ) throws BeansException，IllegalStateException {
	synchronized (this.startupShutdownMonitor) {
		//准备刷新的上下文环境
		prepareRefresh () ;
		// Tell the subclass to refresh the internal bean factory.
        //初始化BeanFactory，并进行XML文件读取
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory() ;
		// Prepare the bean factory for use in this context.
		//对 BeanFactory进行各种功能填充，如对@Autowired的支持等
		prepareBeanFactory (beanFactory) ;
		try {
			// Allows post-processing of the bean factory in context subclasses.
       	    //子类覆盖方法做额外的处理
			postProcessBeanFactory (beanFactory);
            //激活各种 BeanFactory处理器
			invokeBeanFactoryPostProcessors (beanFactory) ;
			//注册拦截Bean创建的Bean处理器，这里只是注册，真正的调用是在getBean时候			
            registerBeanPostProcessors(beanFactory) ;
			//为上下文初始化Message 源，即不同语言的消息体，国际化处理
            initMessagesource () ;
			// Initialize event multicaster for this context.
			//初始化应用消息广播器，并放入“applicationEventMulticaster" bean中		
          	 initApplicationEventMulticaster () ;
			//Initialize other special beans in specific context subclass
    		//留给子类来初始化其它的 Bean
			onRefresh () ;
			// check for listener beans and register them.
			//在所有注册的bean中查找Listener bean，注册到消息广播器中
            registerListeners() ;
			// Instantiate all remaining (non-lazy-init) singletons
            //初始化剩下的单实例（非惰性的)
			finishBeanFactoryInitialization (beanFactory) ;
			//Last step: publish corresponding event.
			//完成刷新过程，通知生命周期处理器lifecycleProcessor刷新过程,
			//同时发出ContextRefreshEvent通知别人
			finishRefresh () ;
        }
        catch (BeansException ex) {
			// Destroy already created singletons to avoid dangling resources
            destroyBeans ();
			// Reset 'active' flag.
            cance1Refresh(ex);
			// Propagate exception to caller.
            throw ex;
        }
    }
}
```

# postProcessBeanFactory

具体功能看“IOC原理”部分。

当实现多个BeanFactorypostProcessor时，需要额外实现Order接口，以指定执行顺序。

注意：BeanFactorypostProcessor也是一个bean，具体配置看“IOC原理”部分。

```
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory){
	// Invoke BeanDefinitionRegistryPostProcessors first, if any.
	Set<String> processedBeans = new HashSet<String> ();
	//对BeanDefinitionRegistry类型的处理
	if (beanFactory instanceof BeanDefinitionRegistry) {
		BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
		List<BeanFactoryPostProcessor> regularPostProcessors = new LinkedList
<BeanFactoryPostProcessor> () ;
        /**
        BeanDefinitionRegistryPostProcessor
        */
        List<BeanDefinitionRegistryPostProcessor> registryPostProcessors = new
LinkedList<BeanDefinitionRegistryPostProcessor>();
        /*
		*硬编码注册的后处理器
        */
		for (BeanFactoryPostProcessor postProcessor ; getBeanFactoryPostProcessors()){
			if (postProcessor instanceof BeanDefinitionRegistryPostProcessor){
				BeanDefinitionRegistryPostProcessor registryPostProcessor =(Bean
DefinitionRegistryPostProcessor) postProcessor;
				//对于BeanDefinitionRegistryPostProcessor类型，在BeanFactoryPostProcessor
				//的基础上还有自己定义的方法，需要先调用
				registryPostProcessor.postProcessBeanDefinitionRegistry(registry);
                registryPostProcessors.add (registryPostProcessor);
			}
            else {
				//记录常规BeanFactoryPostProcessor
				regularPostProcessors.add (postProcessor);
            }
        }
        /*
		配置注册的后处理器
        */
		Map<String,BeanDefinitionRegistryPostProcessor> beanMap = beanFactory.
getBeansofType(BeanDefinitionRegistryPostProcessor.class,true，false) ;
		List<BeanDefinitionRegistryPostProcessor> registryPostProcessorBeans
new ArrayList<BeanDefinitionRegistryPostProcessor>(beanMap.values() );
		orderComparator.sort (registryPostProcessorBeans) ;
		for(BeanDefinitionRegistryPostProcessor postProcessor : registryPostProcessorBeans)			{
			//BeanDefinitionRegistryPostProcessor的特殊处理
			postProcessor.postProcessBeanDefinitionRegistry (registry);
		 }
		//激活postProcessBeanFactory方法，之前激活的是
        //postProcessBeanDefinitionRegistry硬编码设置的BeanDefinitionRegistryPostProcessor
		invokeBeanFactoryPostProcessors (registryPostProcessors，beanFactory);
        //配置的BeanDefinitionRegistryPostProcessor  说几把啥呢这注释
		invokeBeanFactoryPostProcessors (registryPostProcessorBeans，beanFactory) ;
        //常规BeanFactoryPostProcessor
		invokeBeanFactoryPostProcessors(regularPostProcessors，beanFactory);
        processedBeans.addAll(beanMap.keyset()) ;
    }
	else {
		//Invoke factory processors registered with the context instance.
		invokeBeanFactoryPostProcessors(getBeanFactoryPostProcessors ()，beanFactory);
    }
	//对于配置中读取的 BeanFactoryPostProcessor的处理
	string[] postProcessorNames = beanFactory.getBeanNamesForType(BeanFactorypost
Processor.class, true, false) ;
	List<BeanFactoryPostProcessor> priority0rderedPostProcessors = new ArrayList
<BeanFactoryPostProcessor>();
	List<String> orderedPostProcessorNames = new ArrayList<String>();
	List<String> nonOrderedPostProcessorNames = new ArrayList<String>();
	//对后处理器进行分类
	for (String ppName : postProcessorNames) {
        if (processedBeans.contains (ppName) ) {
			//已经处理过
		}
        else if (isTypeMatch(ppName，Priorityordered.class)){
			priorityorderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.ciass));
		}
        else if (isTypeMatch(ppName,ordered.class)) {
			orderedPostProcessorNames.add(ppName) ;
		}
        else {
			nonorderedPostProcessorNames.add (ppName) ;
        }
    }
	//按照优先级进行排序
	orderComparator.sort(priorityorderedPostProcessors) ;
	invokeBeanFactoryPostProcessors(priorityorderedPostProcessors，beanFactory);
	//Next,invoke the BeanFactoryPostProcessors that implement Ordered.
	List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<BeanFactory
PostProcessor>();
	for (String postProcessorName : orderedPostProcessorNames){
		orderedPostProcessors.add (getBean (postProcessorName，BeanFactoryPostProcessor.
class));
	}
	//按照order排序
	orderComparator.sort (orderedPostProcessors) ;
	invokeBeanFactoryPostProcessors(orderedPostProcessors,beanractory) ;
	//无序，直接调用
	List<BeanFactoryPostProcessor> nonrderedPostProcessors = new ArrayList<BeanFactory
PostPrccessor>();
	for (string postProcessorName : nonOrderedPostProcessorNames) {
		nonOrderedPostProcessors.add (getBean (postProcessorName,
BeanFactoryPostProcessor.class) ) ;
    }
	invokeBeanFactoryPostProcessors(nonorderedPostProcessors，beanFactory);
}
```

从上面的方法中我们看到，对于BeanFactoryPostProcessor 的处理主要分两种情况进行：一个是对于BeanDefinitionRegistry类的特殊处理，另一种是对普通的BeanFactoryPostProcessor进行处理。而对于每种情况都需要考虑硬编码注入注册的后处理器以及通过配置注入的后处理器。

**对于BeanDefinitionRegistry类型的处理类的处理主要包括以下内容。：**

1.  对于硬编码注册的后处理器的处理，主要是通过AbstractApplicationContext中的添加处理器方法addBeanFactoryPostProcessor进行添加。添加后的后处理器会存放在beanFactoryPostProcessors中，而在处理BeanFactoryPostProcessor时候会首先检测beanFactoryPostProcessors是否有数据。当然,BeanDefinitionRegistryPostProcessor继承自BeanFactoryPostProcessor，不但有BeanFactoryPostProcessor的特性，同时还有自己定义的个性化方法，也需要在此调用。所以，这里需要从 beanFactoryPostProcessors中挑出BeanDefinitionRegistryPostProcessor 的后处理器,并进行其postProcessBeanDefinitionRegistry方法的激活。
1.  记录后处理器主要使用了三个List完成。

-   -   registryPostProcessors: 记录通过**硬编码方式**注册的 BeanDefinitionRegistryPostProcessor类型的处理器。
    -   regularPostProcessors: 记录通过**硬编码方式**注册的BeanFactoryPostProcessor类型的处理器。

<!---->

-   -   registryPostProcessorBeans: 记录通过**配置方式**注册的BeanDefinitionRegistryPostProcessor类型的处理器。

3.  对以上所记录的 List中的后处理器进行统一调用BeanFactoryPostProcessor的postProcessBeanFactory方法。有些特殊的后处理会在加入List之前就调用
3.  对beanFactoryPostProcessors 中非 BeanDefinitionRegistryPostProcessor类型的后处理器进行统一的 BeanFactoryPostProcessor 的 postProcessBeanFactory方法调用。


**普通beanFactory处理：**

BeanDefinitionRegistryPostProcessor只对BeanDefinitionRegistry类型的ConfigurableListableBeanFactory有效，所以如果判断所示的 beanFactory并不是 BeanDefinitionRegistry，那么便可以忽略 BeanDefinitionRegistryPostProcessor，而直接处理 BeanFactoryPostProcessor，当然获取的方式与上面的获取类似。

这里需要提到的是，对于硬编码方式手动添加的后处理器是不需要做任何排序的，但是在配置文件中读取的处理器，Sping并不保证读取的顺序。所以，为了保证用户的调用顺序的要求，Spring对于后处理器的调用支持按照PriorityOrdered或者Ordered的顺序调用。

# 后处理

## 注册BeanPostProcessor

这里并不是调用，而是注册。真正的调用其实是在 bean的初始化前后阶段进行的。

ApplicationContext使用registerBeanPostProcess方法注册BeanPostProcessor

```
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory){
	string[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class,
true, false);
    /*
    BeanPostProcessorChecker是一个普通的信息打印，可能会有些情况，当Spring的配置中的后处理器还没有被注册就已经开始了bean 的初始化时便会打印出 BeanPostProcessorChecker中设定的信息
    */
	int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount()+1+
postProcessorNames.length;
	beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory,
beanProcessorTargetCount) );
	//使用Priorityordered保证顺序
	List<BeanPostProcessor>priority0rderedPostProcessors = new ArrayList<Bean
PostProcessor> ();
	//MergedBeanDefinitionPostProcessor
	List<BeanPostProcessor>internalPostProcessors = new ArrayList<BeanPost
Processor>();
	//使用ordered保证顺序
	List<String> orderedPostProcessorNames = new ArrayList<String>();
	//无序BeanPostProcessor
	List<String> nonorderedPostProcessorNames = new ArrayList<String>();
	for (string ppName : postProcessorNames) {
		if (isTypeMatch (ppName，Priorityordered.class)){
			BeanPostProcessor pp = beanFactory.getBean (ppName，BeanPostProcessor.class);
            priorityorderedPostProcessors.add (pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor){
           	 internalPostProcessors.add(pp);
			}
		}
   		else if (isTypeMatch (ppName,Ordered.class)) {
			orderedPostProcessorNames. add(ppNae) ;
		}
    	else {
			nonorderedPostProcessorNames.add (ppName) ;
    	}
    }
    //第一步，注册所有实现Priorityordered的BeanPostProcessor
    //ordercomparator.sort(priorityorderedPostProcessors);
	//registerBeanPostProcessors(beanFactory，priorityorderedPostProcessors) ;
    
	//第二步，注册所有实现Ordered的 BeanPostProcessor
	List<BeanPostProcessor> orderedPostProcessors = new ArrayList<BeanPostProcessor>();
    for (String ppName : orderedPostProcessorNames){
		BeanPostProcessor pp = beanFactory.getBean (ppName，BeanPostProcessor.class);
        orderedPostProcessors.add (pp);
		if (pp instanceof MergedBeanDefinitionPostProcessor){
			internalPostProcessors.add (pp);
        }
    }
	orderComparator.sort (orderedPostProcessors) ;
	registerBeanPostProcessors(beanFactory，orderedPostProcessors);
    
	//第三步，注册所有无序的BeanPostProcessor
	List<BeanPostProoessor> nonOrderedPostProcessors = new ArrayList<BeanPostProcessor>();
    for (string ppName : nonorderedPostProcessorNames) {
		BeanPostProcessor pp = beanFactory.getBean (ppName，	BeanPostProcessor.class);
        nonOrderedPostProcessors.add (pp) ;
		if (pp instanceof MergedBeanDefinitionPostProcessor) {
            internalPostProcessors.add(pp) ;
        }
    }
	registerBeanPostProcessors(beanFactory，nonOrderedPostProcessors) ;
    //第四步，注册所有MergedBeanDefinitionPostProcessor类型的BeanPostProcessor，并非
	//重复注册，在beanFactory.addBeanPostProcessor中会先移除已经存在的BeanPostProcessor
    orderComparator.sort (internalPostProcessors) ;
	registerBeanPostProcessors (beanFactory， internalpostProcessors);
	//添加ApplicationListener探测器
	beanFactory.addBeanPostProcessor(new applicationListenerDetector() ) ;
}
```

首先我们会发现，对于 BeanPostProcessor的处理与 BeanFactoryPostProcessor的处理极为相似，但是似乎又有些不一样的地方。经过反复的对比发现，对于 BeanFactoryPostProcessor的处理要区分两种情况，一种方式是通过硬编码方式的处理，另一种是通过配置文件方式的处理。那么为什么在 BeanPostProcessor的处理中只考虑了配置文件的方式而不考虑硬编码的方式呢？提出这个问题，还是因为读者没有完全理解两者实现的功能。

对于BeanFactoryPostProcessor的处理，不但要实现注册功能，而且还要实现对后处理器的激活操作，所以需要载入配置中的定义，并进行激活；

而对于BeanPostProcessor并不需要马上调用，再说，硬编码的方式实现的功能是将后处理器提取并调用，这里并不需要调用，当然不需要考虑硬编码的方式了，这里的功能只需要将配置文件的BeanPostProcessor提取出来并注册进入 beanFactory就可以了。对于beanFactory 的注册，也不是直接注册就可以的。在Spring中支持对于 BeanPostProcessor的排序,比如根据PriorityOrdered进行排序，根据Ordered进行排序或者无序，而Spring在 BeanPostProcessor 的激活顺序的时候也会考虑对于顺序的问题而先进行排序。

## 国际化信息支持

看“IOC原理”部分



## 容器内事件发布

看“IOC原理”部分


